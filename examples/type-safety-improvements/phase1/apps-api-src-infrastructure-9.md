# Type-Safety Audit: apps-api-src-infrastructure-9

**Batch 61** — `apps/api/src/infrastructure/rate-limiting/` and `apps/api/src/infrastructure/webhooks/`

## Summary

7 files reviewed. Most are well-typed. The main issues are Redis result type casts required by the ioredis API surface and a duck-typing pattern in the retry classifier that uses `(obj as Record<string, unknown>)` for error introspection. Low actionable surface overall.

---

## Findings

### 1

- **File**: `apps/api/src/infrastructure/rate-limiting/atomic-consume.service.ts`
- **Line**: 84
- **Category**: Type casts
- **Impact**: Low
- **Description**: `(await this.redis.script("LOAD", luaScript)) as string` — ioredis types `script()` as returning `unknown`; the cast is necessary given the SDK surface but could be narrowed.
- **Suggestion**: Wrap the call in a helper that validates `typeof result === "string"` before returning, eliminating the blind cast.
- **Evidence**: `const sha256 = (await this.redis.script("LOAD", luaScript)) as string;`

---

### 2

- **File**: `apps/api/src/infrastructure/rate-limiting/atomic-consume.service.ts`
- **Lines**: 164–168, 284–288
- **Category**: Type casts
- **Impact**: Low–Medium
- **Description**: `(await this.redis.evalsha(...)) as [number, number, number]` — Lua script return values are typed as `unknown` by ioredis; the tuple shape is assumed, not verified. If the script changes, the cast becomes silently wrong.
- **Suggestion**: Parse with `z.tuple([z.number(), z.number(), z.number()])` and throw on parse failure, making any script change detectable.
- **Evidence**: `const [tokensRemaining, requestsRemaining, retryAfterMs] = (await this.redis.evalsha(...)) as [number, number, number];`

---

### 3

- **File**: `apps/api/src/infrastructure/rate-limiting/retry-classifier.ts`
- **Lines**: ~400–430
- **Category**: `Record<string, ...>` / Type casts
- **Impact**: Low
- **Description**: Internal helpers `extractRecord`, `extractArray`, `extractNumber`, `extractString` all use `(obj as Record<string, unknown>)[key]` to duck-type SDK error shapes. This is intentional for cross-provider error introspection but the cast is repeated rather than centralised.
- **Suggestion**: Extract a single `asRecord(v: unknown): Record<string, unknown>` guard function and call it from all four helpers.
- **Evidence**: Pattern repeated across helpers: `(obj as Record<string, unknown>)[key]`

---
