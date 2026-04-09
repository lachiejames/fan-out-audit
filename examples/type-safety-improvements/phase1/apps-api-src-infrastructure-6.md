# Audit: apps-api-src-infrastructure-6

**Files inspected**: 8
**Findings**: 5

## Summary

This batch covers the logger service, logger utilities, metrics service, outbound HTTP interceptor, request ID interceptor, timer utilities, Sentry business events, and the account-deletion queue service. These files are generally clean and minimal. Findings centre on: (1) `MetricPayload` uses an index signature alongside explicit keys, making the explicit keys weaker than intended; (2) `sanitizeObject` in `logger.service.ts` and `sanitizeLogData` in `logger.utils.ts` duplicate the same sensitive-field redaction logic; (3) the `globalThis` declaration in `outbound-http.interceptor.ts` uses a `var` ambient declaration which is less safe than a proper `declare global` augmentation.

## Findings

### Finding 1: `MetricPayload` index signature weakens typed keys

- **File**: `apps/api/src/infrastructure/logging/metrics.service.ts:43`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `MetricPayload` declares named fields (`event`, `userId`, `platform`, `durationMs`, `count`, `tokens`) but also has `[key: string]: unknown`. The index signature causes TypeScript to treat all named keys as `unknown` for read access, meaning a caller that reads `payload.event` gets `MetricEvent | undefined` rather than the required `MetricEvent`. It also allows passing payloads that omit `event` or `userId` without a compile error (since the index signature satisfies the interface shape check in some contexts).
- **Suggestion**: Remove the `[key: string]: unknown` index signature. If extra fields are needed, use a union or intersection: `MetricPayload & Record<string, unknown>` only at the `track()` call site.
- **Evidence**:

```typescript
export interface MetricPayload {
  event: MetricEvent;
  userId: string;
  platform?: string;
  durationMs?: number;
  count?: number;
  tokens?: number;
  [key: string]: unknown; // weakens all typed keys above
}
```

### Finding 2: Duplicate sensitive-field redaction logic across `logger.service.ts` and `logger.utils.ts`

- **File**: `apps/api/src/infrastructure/logging/logger.service.ts:38–49` and `apps/api/src/infrastructure/logging/logger.utils.ts:25–37`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `AppLoggerService` defines a private `SENSITIVE_PATTERNS` constant and a private `sanitizeObject` method. `logger.utils.ts` exports `sanitizeLogData` which contains its own copy of similar (but not identical) patterns — `logger.utils.ts` additionally includes `accessToken` and `refreshToken` patterns that `logger.service.ts` omits. The two implementations can drift independently.
- **Suggestion**: Consolidate into a single exported `sanitizeLogData` from `logger.utils.ts` and have `AppLoggerService.sanitizeObject` delegate to it. Ensure the pattern sets are merged first.
- **Evidence**:

```typescript
// logger.service.ts:38 - missing accessToken/refreshToken
const SENSITIVE_PATTERNS = [/password/i, /token/i, /secret/i, /apiKey/i, /api_key/i, /authorization/i, /cookie/i, /sessionId/i, /session_id/i];

// logger.utils.ts:25 - has accessToken/refreshToken
const sensitivePatterns = [..., /accessToken/i, /refreshToken/i];
```

### Finding 3: `globalThis` augmentation uses unsafe `var` declaration

- **File**: `apps/api/src/infrastructure/logging/outbound-http.interceptor.ts:25`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The `declare global { var __slopweaverFetchIntercepted: boolean | undefined; }` ambient declaration uses `var` in the global scope. This is the required syntax for `globalThis` augmentation in TypeScript, but the property type `boolean | undefined` allows the value to be set to `undefined` which is semantically the same as absent. Using `boolean` (with `globalThis.__slopweaverFetchIntercepted ?? false` at the read site) would be more precise.
- **Suggestion**: Change the declaration to `var __slopweaverFetchIntercepted: boolean` and update the read site to `globalThis.__slopweaverFetchIntercepted ?? false`.
- **Evidence**:

```typescript
declare global {
  var __slopweaverFetchIntercepted: boolean | undefined;
}
```

### Finding 4: `TimerContext` index signature weakens the required `msg` key

- **File**: `apps/api/src/infrastructure/logging/timer.utils.ts:23`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `TimerContext` requires `msg: string` but also includes `[key: string]: unknown`. For the same reason as Finding 1 (`MetricPayload`), the index signature causes TypeScript to allow `msg` to be typed as `unknown` when read back from a `TimerContext` variable (though in practice `msg` is always written correctly).
- **Suggestion**: Consider whether the index signature is truly necessary. If arbitrary extra fields are needed in timer logs, use `{ msg: string } & Record<string, unknown>` at the `end()` call site without polluting the interface itself.
- **Evidence**:

```typescript
export interface TimerContext {
  msg: string;
  [key: string]: unknown;
}
```

### Finding 5: `sanitizeLogData` generic return type uses unsafe `as T` cast

- **File**: `apps/api/src/infrastructure/logging/logger.utils.ts:53`
- **Category**: type-cast
- **Impact**: low
- **Description**: `sanitizeLogData<T>` accepts `T` and returns `T`. The implementation constructs `sanitized: Record<string, unknown>` then casts it back to `T` with `return sanitized as T`. This is structurally incorrect: `sanitized` is always `Record<string, unknown>` regardless of what `T` was. Passing `T = { accessToken: string }` returns a value typed as `{ accessToken: string }` but the `accessToken` key has been replaced with `"[REDACTED]"` (a string), so the structural contract is maintained only by coincidence.
- **Suggestion**: Change the return type to `Record<string, unknown>` (or a more constrained sanitized type) and update call sites to accept that type. The generic parameter gives a false sense of type preservation.
- **Evidence**:

```typescript
export function sanitizeLogData<T>({ data }: { data: T }): T {
  // ...
  return sanitized as T; // 'sanitized' is Record<string, unknown>, not T
}
```
