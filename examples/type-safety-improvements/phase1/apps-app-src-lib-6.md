# Type-Safety Audit: apps-app-src-lib-6

**Files inspected**: 8
**Findings**: 3

## Summary

`prefetch-chunks.ts` uses `Record<string, ChunkLoader>` where `Partial<Record<PlatformId, ChunkLoader>>` would be safe and narrower. `rate-limit-store.ts` has two inline `as Record<string, unknown>` casts in an error-body parser that could be replaced with type guards. `persist/search-store.ts`, `persist/triage-store.ts`, `query-retry.ts`, `posthog.ts`, and `recent-calendar-executions-store.ts` are clean.

---

## Findings

### 1

**File**: `apps/app/src/lib/prefetch-chunks.ts`
**Category**: `Record<string, ...>` where stricter types exist
**Impact**: Low — runtime behavior unchanged; but a misspelled platform ID would compile silently
**Description**: `platformChunkLoaders: Record<string, ChunkLoader>` at line 41. `PlatformId` is imported from `@slopweaver/contracts` in the same file. Switching to `Partial<Record<PlatformId, ChunkLoader>>` would make TypeScript reject unknown platform keys at compile time.
**Suggestion**: Change the declaration to `platformChunkLoaders: Partial<Record<PlatformId, ChunkLoader>>` and fix any initializer entries that use non-`PlatformId` keys.
**Evidence**: `const platformChunkLoaders: Record<string, ChunkLoader> = { ... }` — line 41

---

### 2

**File**: `apps/app/src/lib/rate-limit-store.ts`
**Category**: Type casts (`as SomeType`) that could be avoided
**Impact**: Low — the casts are guarded by `typeof` / nullability checks, so they are safe in practice
**Description**: `parseRateLimitBlockedBody` casts `body as Record<string, unknown>` (line 108) and `details as Record<string, unknown>` (line 113) in order to access unknown properties. A narrow type guard would be more explicit and reusable.
**Suggestion**: Extract `isRecord(v: unknown): v is Record<string, unknown>` and use it before accessing properties, or use optional chaining on a Zod parse result.
**Evidence**:

```typescript
const bodyRecord = body as Record<string, unknown>; // line 108
const d = details as Record<string, unknown>; // line 113
```

---

### 3

**File**: `apps/app/src/lib/utils/classify-error.ts`
**Category**: Type casts (`as SomeType`) that could be avoided
**Impact**: Low — the casts are guarded by `instanceof` checks; but the same pattern is repeated three times
**Description**: Three inline `as` casts at lines 108, 123, and 298 extract `status`, `body`, and `code` from an `unknown` error. The pattern `(error as { status: number }).status` could be replaced with a single `isHttpError(e: unknown): e is { status: number; body?: unknown }` type guard.
**Suggestion**: Add an `isHttpError` type predicate in the same file. This removes all three casts and makes the error-shape contract explicit.
**Evidence**:

- Line 108: `const { status } = error as { status: number }`
- Line 123: `(error as { body?: unknown }).body`
- Line 298: `(body as Record<string, unknown>)["code"]`
