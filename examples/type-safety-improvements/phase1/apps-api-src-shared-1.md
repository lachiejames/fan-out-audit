# Audit: apps-api-src-shared-1

**Files inspected**: 8
**Findings**: 4

## Summary

The AI utility and cache infrastructure files are generally well-typed, but `ai-usage.utils.ts` contains several unsafe `as` casts on `unknown` objects that could be replaced with structured type guards. The `cache.port.ts` file uses `unknown` return types that are an expected design choice but still carry minor risk at call sites. The `outbound-rate-limits.config.ts` has a `Record<OutboundProvider, ...>` built via `Object.fromEntries` with a manual cast that could be made safer.

## Findings

### Finding 1: Unsafe `as Record<string, unknown>` cast on usage object

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/ai/utils/ai-usage.utils.ts:46`
- **Category**: type-cast
- **Impact**: medium
- **Description**: After the `typeof usage !== "object"` guard, the code casts `usage` to `Record<string, unknown>` using `as`. While this is a valid pattern for handling unknown SDK payloads, it bypasses TypeScript's structural checks. Any property access on `u` is unchecked. A tighter approach is possible.
- **Suggestion**: Define a local type guard `function isObjectRecord(v: unknown): v is Record<string, unknown>` and use it instead of the inline cast on line 46. The existing guard on line 43 already verifies `typeof usage === "object"` and non-null, so the cast is reliable in practice, but a named predicate communicates intent and is reusable.
- **Evidence**:

```typescript
const u = usage as Record<string, unknown>;
```

### Finding 2: Nested `as Record<string, unknown>` cast for cache creation detail

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/ai/utils/ai-usage.utils.ts:74`
- **Category**: type-cast
- **Impact**: low
- **Description**: A second `as Record<string, unknown>` cast is applied to the nested `nestedCacheCreation` object after an `typeof === "object"` guard. Same pattern as Finding 1 — safe in practice but a named type guard would be cleaner.
- **Suggestion**: Reuse the same type guard from Finding 1.
- **Evidence**:

```typescript
const nested = nestedCacheCreation as Record<string, unknown>;
```

### Finding 3: `Record<OutboundProvider, ProviderRateLimitConfig>` built with unsafe cast from `Object.fromEntries`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/config/outbound-rate-limits.config.ts:525-527`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `Object.fromEntries` returns `Record<string, ProviderRateLimitConfig>` which is then cast to `Record<OutboundProvider, ProviderRateLimitConfig>`. If a config entry is ever added to the array without being in the `OutboundProvider` union, the cast silently lies. The array is `as const satisfies readonly ProviderRateLimitConfig[]` above, so all providers are known, but the `fromEntries` cast is still a manual assertion.
- **Suggestion**: Validate at the `satisfies` boundary that all `OutboundProvider` values are present. Alternatively, keep the cast but wrap it with a compile-time check using `satisfies Record<OutboundProvider, ProviderRateLimitConfig>` on the result rather than a bare `as`.
- **Evidence**:

```typescript
export const OUTBOUND_RATE_LIMITS: Record<OutboundProvider, ProviderRateLimitConfig> = Object.fromEntries(
  OUTBOUND_RATE_LIMIT_CONFIGS.map((cfg) => [cfg.provider, cfg]),
) as Record<OutboundProvider, ProviderRateLimitConfig>;
```

### Finding 4: `endpointClasses` typed as `Record<string, EndpointClassConfig>` where a stricter union key would be safer

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/config/outbound-rate-limits.config.ts:118`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `ProviderRateLimitConfig.endpointClasses` is typed as `Record<string, EndpointClassConfig>` but usage in the file shows only `"read"` and `"write"` as keys. Any string key is accepted, which widens the type unnecessarily and removes autocomplete for callers.
- **Suggestion**: Change to `Partial<Record<"read" | "write", EndpointClassConfig>>` or a more precise union if more classes are expected. If the key set is truly open, document why.
- **Evidence**:

```typescript
endpointClasses?: Record<string, EndpointClassConfig>;
```
