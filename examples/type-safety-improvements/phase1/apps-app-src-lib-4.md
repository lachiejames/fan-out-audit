# Type-Safety Audit: apps-app-src-lib-4

**Files inspected**: 8
**Findings**: 3

## Summary

`lazy-retry.ts` uses `ComponentType<any>` in a generic constraint, which can be tightened. `paywall-event.ts` has two fields typed as `unknown` that are set after a Zod parse and could be typed more precisely. `persist/legacy-persist-storage.ts` has a necessary-but-improvable `as` cast for the storage envelope. The remaining files (`errors/api-response-error.ts`, `errors/api-timeout-error.ts`, `native-notifications/orchestrator.ts`, `native-notifications/utils.ts`, `onboarding-context.tsx`) are clean.

---

## Findings

### 1

**File**: `apps/app/src/lib/lazy-retry.ts`
**Category**: `any` in production code
**Impact**: Low — the `any` is in a generic constraint, not a return value; real component props remain typed at call-sites
**Description**: `ComponentType<any>` appears at line 17 as the upper bound for the generic parameter `T`. This allows passing truly any component but disables checking of prop types when the result is assigned.
**Suggestion**: Replace with `ComponentType<Record<string, unknown>>` or simply `ComponentType` (which defaults to `ComponentType<{}>` in React 18+), both of which give the same flexibility without explicitly opening up `any`.
**Evidence**: `function lazyWithRetry<T extends ComponentType<any>>` — line 17

---

### 2

**File**: `apps/app/src/lib/paywall-event.ts`
**Category**: `any` or unsafe `unknown` in production code
**Impact**: Low — the `unknown` fields are set inside a Zod `.safeParse()` success branch, so they could carry precise types
**Description**: `PaywallEventDetail.message?: unknown` and `PaywallEventDetail.statusCode?: unknown` (lines 7–8) are set from the parsed error body after a Zod schema succeeds. Since the schema defines the shape of `message` (string array) and `statusCode` (number), the interface fields could be narrowed to match the schema inferred types.
**Suggestion**: Change `message?: unknown` to `message?: string[]` and `statusCode?: unknown` to `statusCode?: number`, consistent with the `StandardErrorSchema` used elsewhere.
**Evidence**:

```typescript
interface PaywallEventDetail {
  message?: unknown;    // line 7
  statusCode?: unknown; // line 8
```

---

### 3

**File**: `apps/app/src/lib/persist/legacy-persist-storage.ts`
**Category**: Type casts (`as SomeType`) that could be avoided
**Impact**: Low — the cast is correct for JSON deserialization but could be replaced with a type predicate
**Description**: Line 81 parses localStorage JSON and then casts `parsed as { state?: unknown; version?: number }`. This inline cast silently accepts malformed data. A type guard would make the narrowing explicit and testable.
**Suggestion**: Extract `isStorageEnvelope(v: unknown): v is { state?: unknown; version?: number }` as a type predicate and use it instead of the cast.
**Evidence**: `const envelope = parsed as { state?: unknown; version?: number };` — line 81
