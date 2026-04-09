# Audit: apps-api-src-infrastructure-3

**Files inspected**: 8
**Findings**: 6

## Summary

This batch covers PostHog service, cache infrastructure, HTTP logging middleware, and serialization utilities. The most significant findings are: (1) a structural type cast on the `cache-manager` store to access a non-standard `deleteByPattern` method; (2) `Record<string, unknown>` log data accumulator in the HTTP logging middleware (acceptable but worth noting); (3) `as const` cast in `serialization.utils.ts` when sorting object keys; and (4) a loose structural cast on the `Cache` object in `cache.adapter.ts` used for store introspection.

## Findings

### Finding 1: Unsafe structural cast on `Cache` to access `deleteByPattern` (cache.adapter.ts)

- **File**: `apps/api/src/infrastructure/cache/cache.adapter.ts:55`
- **Category**: type-cast
- **Impact**: medium
- **Description**: The `deleteByPattern` implementation casts `this.cacheManager` to a hand-written inline type `{ store?: { deleteByPattern?: ... }; stores?: Array<...> }` to access a non-standard extension of the `cache-manager` `Cache` interface. If the underlying cache-manager version changes its internal shape, this will silently do nothing (the guard `!store?.deleteByPattern` is already there, but the cast itself obscures what is actually expected).
- **Suggestion**: Define a named interface `CacheManagerWithStore` and export it alongside the adapter, or extend the `Cache` type via module augmentation so the cast is centralised and documented.
- **Evidence**:

```typescript
const manager = this.cacheManager as {
  store?: { deleteByPattern?: (pattern: string) => Promise<void> };
  stores?: Array<{ deleteByPattern?: (pattern: string) => Promise<void> }>;
};
```

### Finding 2: `CacheKey` cast to `string` on every store call

- **File**: `apps/api/src/infrastructure/cache/cache.adapter.ts:25,36,44,64`
- **Category**: type-cast
- **Impact**: low
- **Description**: `CacheKey` is a branded type that cannot be assigned to `string` without an explicit cast. Every call to `cacheManager.get/set/del/deleteByPattern` requires `key as string` or `pattern as string`. If `CacheKey` is always a `string` at runtime, the type should either extend `string` or the `CachePort` interface should expose `string` parameters directly, removing the need for repeated casts.
- **Suggestion**: If `CacheKey` is always a `string` brand, consider `type CacheKey = string & { readonly _brand: 'CacheKey' }` which is assignable to `string` directly, eliminating the casts.
- **Evidence**:

```typescript
async get(key: CacheKey): Promise<unknown | undefined> {
  return this.cacheManager.get(key as string);
}
async set(key: CacheKey, value: unknown, ttlMs: number): Promise<void> {
  await this.cacheManager.set(key as string, value, ttlMs);
}
```

### Finding 3: `Record<string, unknown>` log data object in HTTP logging middleware

- **File**: `apps/api/src/infrastructure/common/middleware/http-logging.middleware.ts:169`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `logData` is typed as `Record<string, unknown>` even though its base keys are always the same fixed set. This prevents TypeScript from catching typos in key names like `logData["requestBody"]`. An interface for the standard HTTP log payload would be safer.
- **Suggestion**: Define a `HttpLogPayload` interface with the known base keys typed explicitly, then use `Partial<HttpLogPayload> & Record<string, unknown>` for the dynamic additions.
- **Evidence**:

```typescript
const logData: Record<string, unknown> = {
  duration,
  ip: req.ip,
  method,
  msg: `${method} ${originalUrl} ${statusCode} - ${duration}ms`,
  requestId,
  statusCode,
  url: originalUrl,
  userAgent: req.headers["user-agent"],
  userId,
};
```

### Finding 4: `obj as Record<string, unknown>` casts in `serialization.utils.ts`

- **File**: `apps/api/src/infrastructure/common/utils/serialization.utils.ts:43,46`
- **Category**: type-cast
- **Impact**: low
- **Description**: `sortObjectKeys` receives `obj: unknown` and must cast it twice: `Object.keys(obj as Record<string, unknown>)` and `(obj as Record<string, unknown>)[key]`. These casts are safe because they are guarded by `typeof obj === "object"`, but TypeScript does not narrow `unknown` to `object` in a way that allows index access without the cast.
- **Suggestion**: Add a local type-narrowing variable: `const record = obj as Record<string, unknown>` immediately after the `typeof` check, and use `record` for all subsequent accesses. This keeps the cast in one place and makes the intent clear.
- **Evidence**:

```typescript
if (typeof obj === "object") {
  const sorted: Record<string, unknown> = {};
  const keys = Object.keys(obj as Record<string, unknown>).toSorted();
  for (const key of keys) {
    sorted[key] = sortObjectKeys({ obj: (obj as Record<string, unknown>)[key] });
  }
  return sorted;
}
```

### Finding 5: `serializeProfileForCache` accepts `Record<string, unknown>` but callers pass typed objects

- **File**: `apps/api/src/infrastructure/cache/prompt-cache.service.ts:189`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `serializeProfileForCache({ profile }: { profile: Record<string, unknown> })` accepts any object. Since this is called with a specific `CoreProfile` type, the parameter type could be `Record<string, unknown>` (acceptable for deterministic serialization) but there is no JSDoc constraint or type check verifying the input is serializable. Non-serializable values (e.g. `Date`, `undefined`) would silently produce `null` or be omitted by `JSON.stringify`.
- **Suggestion**: Accept `Record<string, JsonValue>` (where `JsonValue` is `string | number | boolean | null | JsonValue[] | { [key: string]: JsonValue }`) to document the serialization contract at the type level.
- **Evidence**:

```typescript
serializeProfileForCache({ profile }: { profile: Record<string, unknown> }): string {
  return deterministicSerialize({ obj: profile });
}
```

### Finding 6: `PromptCacheService` re-declares types already exported from port

- **File**: `apps/api/src/infrastructure/cache/prompt-cache.service.ts:38-78`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `CacheableContentType`, `CacheStats`, `CacheOptimizedPrompt`, and `PromptParts` are defined in `prompt-cache.service.ts` and also imported and re-exported by `prompt-cache.port.ts` (imported in `prompt-cache.adapter.ts:10-15`). The service defines the canonical shape while the port imports from the service — this reverses the expected dependency direction (the port should define types and the service should implement them). If the service's types ever diverge from the port's types, callers of the port will silently use stale shapes.
- **Suggestion**: Move `CacheableContentType`, `CacheStats`, `CacheOptimizedPrompt`, and `PromptParts` to the domain port (`prompt-cache.port.ts`) as the single source of truth, and have the service import from there, not vice versa.
- **Evidence**:

```typescript
// prompt-cache.service.ts (defines canonical types)
export type CacheableContentType = "system" | "core_profile" | "tools" | "writing_patterns";
export interface CacheStats { ... }
export interface CacheOptimizedPrompt { ... }
export interface PromptParts { ... }

// prompt-cache.adapter.ts imports from the port, which presumably re-exports from service:
import { CacheableContentType, CacheOptimizedPrompt, CacheStats, IPromptCachePort, PromptParts }
  from "@/domain/ports/services/prompt-cache.port";
```
