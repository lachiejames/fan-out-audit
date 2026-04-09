# Type-Safety Audit: apps-app-src-lib-5

**Files inspected**: 8
**Findings**: 2

## Summary

`persist/features-store.ts` stores `string[]` but reads back as `FeatureId[]` via a cast; storing as the narrower type would eliminate the cast. All other files (`persist/analytics-once-store.ts`, `persist/diagnostics-preferences-store.ts`, `persist/observations-store.ts`, `persist/onboarding-store.ts`, `persist/persisted-auth-state.ts`, `paywall-event.ts` already covered in lib-4) are clean or have no material issues beyond what is reported.

---

## Findings

### 1

**File**: `apps/app/src/lib/persist/features-store.ts`
**Category**: Type casts (`as SomeType`) that could be avoided
**Impact**: Low — the cast is safe only as long as no non-`FeatureId` string is ever written to the store
**Description**: `new Set(get().enabledFeatures as FeatureId[])` at line 91 casts from `string[]` to `FeatureId[]`. The underlying Zustand state declares `enabledFeatures: string[]`, which means an invalid feature ID could be persisted and later treated as a valid `FeatureId`.
**Suggestion**: Change the persisted state field type to `FeatureId[]`. On store rehydration, filter through `FEATURE_IDS.includes(v)` to discard any stale string values so existing localStorage data degrades gracefully.
**Evidence**: `new Set(get().enabledFeatures as FeatureId[])` — line 91

---

### 2

**File**: `apps/app/src/lib/posthog-super-properties.ts`
**Category**: Type casts (`as SomeType`) that could be avoided
**Impact**: Low — the cast masks a potential undefined/mismatched environment value
**Description**: `import.meta.env["SLOPWEAVER_ENV"] as PostHogSuperProperties["environment"] | undefined` at line 12 blindly asserts that the raw environment string matches the expected union. An invalid `SLOPWEAVER_ENV` value would pass as a valid `environment` at compile time.
**Suggestion**: Add an explicit narrowing check, e.g.:

```typescript
const rawEnv = import.meta.env["SLOPWEAVER_ENV"];
const environment = rawEnv === "production" || rawEnv === "staging" || rawEnv === "development" ? rawEnv : undefined;
```

Or use a Zod literal union parse.
**Evidence**: `import.meta.env["SLOPWEAVER_ENV"] as PostHogSuperProperties["environment"] | undefined` — line 12
