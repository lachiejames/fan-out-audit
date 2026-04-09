# Audit: apps/app/src/components — Slice 4 (analytics sample data and auth)

**Files inspected**: 8
**Findings**: 4

## Summary

The analytics sample data file is the primary concern: it defines four local interfaces that parallel API response shapes already covered by contract types, creating a maintenance burden. A secondary issue is the `Record<string, ...>` platform breakdown map whose keys should be typed with `PlatformId`.

## Findings

### Finding 1: Four local interfaces duplicating contract response shapes

- **File**: `apps/app/src/components/analytics/sample-data.utils.ts:25-52`
- **Category**: Custom types duplicating SDK/contract types
- **Impact**: high
- **Description**: `SamplePredictionStats`, `SampleTrainingStats`, `SampleChatFeedbackStats`, and `SampleTriageStats` are defined locally with JSDoc comments noting they match response shapes in `useAnalyticsData`. These are structural duplicates of types that should be imported from `@slopweaver/contracts` (or derived from the contract response schema). If the API response shape changes, the sample data interfaces will silently diverge.
- **Suggestion**: Replace each local interface with a type alias derived from the contracts package: e.g. `type SamplePredictionStats = z.infer<typeof predictionStatsResponseSchema>` or import the response type directly. If these are truly UI-only sample/mock types that differ from the API shape, document the divergence explicitly and add a comment linking to the contract type.
- **Evidence**: Four exported interfaces at lines 25-52 with JSDoc noting they mirror API response shapes.

### Finding 2: `buildSamplePlatformBreakdown` return type uses `string` keys

- **File**: `apps/app/src/components/analytics/sample-data.utils.ts:109-118`
- **Category**: `Record<string, ...>` where stricter types would be safer
- **Impact**: medium
- **Description**: The function returns `Record<string, { count: number; percentage: number; responseTime: number }>` where the keys are platform IDs. Using `string` allows callers to look up invalid platform IDs without a compile error.
- **Suggestion**: Import `PlatformId` from `@slopweaver/contracts` and return `Partial<Record<PlatformId, { count: number; percentage: number; responseTime: number }>>`.
- **Evidence**: Return type annotation `Record<string, { count; percentage; responseTime }>` at lines 109-118.

### Finding 3: Auth components using string literals for role/tier checks

- **File**: `apps/app/src/components/` (auth-related files in slice 4)
- **Category**: Custom types duplicating SDK/contract types
- **Impact**: medium
- **Description**: Auth gate components compare against subscription tier and role strings without importing the canonical `SubscriptionTier` or role union types from `@slopweaver/contracts`. This means a tier rename in contracts will not surface as a compile error in the gate logic.
- **Suggestion**: Import `SubscriptionTier` from `@slopweaver/contracts` and use it for all tier comparisons. Same for any role union types exposed by contracts.
- **Evidence**: Tier string literals used in conditional rendering checks in auth gate components.

### Finding 4: Sample data export types not re-exported from a central types file

- **File**: `apps/app/src/components/analytics/sample-data.utils.ts`
- **Category**: Duplicate type definitions
- **Impact**: low
- **Description**: The four sample data interfaces are exported directly from a `.utils.ts` file, meaning consumers import types from a utils file rather than a dedicated types module. Combined with Finding 1, this spreads type definitions across implementation files.
- **Suggestion**: Once the interfaces are aligned with contract types (Finding 1), the local definitions can be removed entirely. If sample-only types are needed, consolidate into `apps/app/src/components/analytics/analytics.types.ts`.
- **Evidence**: `export interface SamplePredictionStats { ... }` etc. in a `.utils.ts` file.
