# Audit: apps-api-src-application-4

**Files inspected**: 8
**Findings**: 5

## Summary

These files implement analytics utilities (readiness gating, timeseries bucketing, trend calculation, platform breakdown, and suggestion-learning metrics) plus auth error types and the domain `AuthUser` interface. The main type-safety issues are: a local `TimeRange` type in `analytics.utils.ts` that duplicates the one already exported from `@slopweaver/contracts`; duplicate `Trend`/`TrendDirection` union types defined twice in the same package when the contracts already export a canonical `Trend`; a gratuitous `as TimeRange` cast that only exists because of that local redefinition; a `Record<string, number>` for `METRIC_THRESHOLDS` where the keys are a known fixed set; and a `Record<string, ...>` return type on `buildPlatformBreakdown` where `PlatformId` from contracts would make the key type safe.

## Findings

### Finding 1: Local `TimeRange` duplicates `@slopweaver/contracts` export

- **File**: `apps/api/src/application/analytics/utils/analytics.utils.ts:18`
- **Category**: duplicate-type
- **Impact**: high
- **Description**: `analytics.utils.ts` defines `type TimeRange = "all" | "day" | "month" | "week" | "year"` locally. The identical type is already exported from `@slopweaver/contracts` (`packages/contracts/src/contracts/analytics/types.ts:29`). The local definition is then immediately used to force a cast (`const range = timeRange as TimeRange;` on line 34) — a cast that is only necessary because the function parameter is `string` instead of the canonical `TimeRange`. The service file (`suggestion-learning-analytics.service.ts`) correctly imports `TimeRange` from `@slopweaver/contracts`, so the pattern already exists; this file just doesn't follow it.
- **Suggestion**: Remove the local `type TimeRange` declaration. Change the `timeRange` parameter of `getStartDate` from `string` to `TimeRange` (imported from `@slopweaver/contracts`). This eliminates the cast entirely and lets TypeScript enforce the exhaustive switch at compile time.
- **Evidence**:

```typescript
// analytics.utils.ts:18-34
type TimeRange = "all" | "day" | "month" | "week" | "year";
// ...
export function getStartDate({ timeRange, now = new Date() }: { timeRange: string; now?: Date | undefined }): Date {
  const range = timeRange as TimeRange;   // cast only needed due to the string param
  switch (range) {
```

---

### Finding 2: Unnecessary `as TimeRange` cast

- **File**: `apps/api/src/application/analytics/utils/analytics.utils.ts:34`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `const range = timeRange as TimeRange` casts away the `string` parameter to the locally-defined union. Because the parameter is typed as `string`, nothing stops callers from passing an invalid value; the cast silences TypeScript rather than enforcing correctness. This is a direct consequence of Finding 1 — fixing that finding eliminates this cast automatically.
- **Suggestion**: Fix Finding 1 (use the `TimeRange` import from `@slopweaver/contracts` as the parameter type). The cast and the intermediate `range` variable both disappear; `switch (timeRange)` works directly.
- **Evidence**:

```typescript
export function getStartDate({ timeRange, now = new Date() }: { timeRange: string; now?: Date | undefined }): Date {
  const range = timeRange as TimeRange;
  switch (range) {
```

---

### Finding 3: `Trend` / `TrendDirection` duplicate the contracts type

- **File**: `apps/api/src/application/analytics/utils/analytics.utils.ts:19` and `apps/api/src/application/analytics/utils/analytics-trend.utils.ts:8`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: Two files in the same directory each independently define a local trend-direction union:
  - `analytics.utils.ts:19` — `type Trend = "down" | "neutral" | "up"` (unexported, used only as return type of `determineTrend`)
  - `analytics-trend.utils.ts:8` — `type TrendDirection = "down" | "neutral" | "up"` (unexported, used as return type of `calculateTrend`)

  `@slopweaver/contracts` already exports `export type Trend = z.infer<typeof trendSchema>` (derived from `z.enum(["up", "down", "neutral"])`). These duplications mean any future change to the canonical enum requires touching three places, and callers cannot rely on the types being assignment-compatible without an explicit cast.

- **Suggestion**: Remove both local type aliases. Import `Trend` from `@slopweaver/contracts` and use it as the return type in both files. Since `analytics-trend.utils.ts` uses the name `TrendDirection`, simply alias on import: `import type { Trend as TrendDirection } from "@slopweaver/contracts"` or rename the local usage sites.
- **Evidence**:

```typescript
// analytics.utils.ts:19
type Trend = "down" | "neutral" | "up";

// analytics-trend.utils.ts:8
type TrendDirection = "down" | "neutral" | "up";

// contracts (canonical source):
// packages/contracts/src/contracts/analytics/schemas.ts:11
export const trendSchema = z.enum(["up", "down", "neutral"]);
// packages/contracts/src/contracts/analytics/types.ts:28
export type Trend = z.infer<typeof trendSchema>;
```

---

### Finding 4: `METRIC_THRESHOLDS` keyed with `Record<string, number>` instead of a literal-keyed type

- **File**: `apps/api/src/application/analytics/utils/analytics-readiness.utils.ts:12`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: `METRIC_THRESHOLDS` is declared as `Record<string, number>` and frozen with `as const`. The keys (`"approvalRate"`, `"chatFeedback"`, etc.) are a fixed, known set. Using `Record<string, number>` means:
  1. TypeScript cannot warn if `calculateReadiness` receives a `metricCounts` map with a misspelled or unknown metric ID.
  2. `METRIC_THRESHOLDS[metricId]` returns `number | undefined` (hence the `?? 1` fallback on line 57), but with a proper literal-keyed type the access would be `number` without the need for the fallback.
  3. `getDesiredDirection` in `analytics-trend.utils.ts` also takes `metricId: string`, so the set of valid IDs is never enforced anywhere in the pipeline.
- **Suggestion**: Extract a `MetricId` union from the object's keys and use it throughout:
  ```typescript
  const METRIC_THRESHOLD_MAP = {
    approvalRate: 10,
    chatFeedback: 1,
    // ...
  } as const;
  export type MetricId = keyof typeof METRIC_THRESHOLD_MAP;
  export const METRIC_THRESHOLDS: Record<MetricId, number> = METRIC_THRESHOLD_MAP;
  ```
  Then type `metricCounts` as `Partial<Record<MetricId, number>>` and `metricId` parameters in `getDesiredDirection` and `DOWN_IS_GOOD_METRICS` as `MetricId`.
- **Evidence**:

```typescript
// analytics-readiness.utils.ts:12-23
export const METRIC_THRESHOLDS: Record<string, number> = {
  approvalRate: 10,
  chatFeedback: 1,
  confidenceMix: 10,
  editFrequency: 10,
  estimatedTimeSaved: 1,
  patternsLearned: 1,
  responseTime: 10,
  trainingCategories: 1,
  triageOverrides: 1,
  volume: 1,
} as const;
// line 57 — forced nullish coalescing because Record<string,number> makes access possibly-undefined:
const threshold = METRIC_THRESHOLDS[metricId] ?? 1;
```

---

### Finding 5: `buildPlatformBreakdown` return type uses `Record<string, ...>` where `PlatformId` is available

- **File**: `apps/api/src/application/analytics/utils/analytics.utils.ts:162-167`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `buildPlatformBreakdown` returns `Record<string, { count: number; percentage: number; responseTime: string | null }>`. The keys are platform identifiers produced by `sourceToPlatform()`, which always returns a platform string. `PlatformId` from `@slopweaver/contracts` is the canonical union for all valid platform identifiers. Using `string` as the key type means callers can look up any arbitrary string without a type error, and the internal `platformCounts` accumulator also uses `Record<string, number>`.
- **Suggestion**: Type the return value as `Partial<Record<PlatformId, { count: number; percentage: number; responseTime: string | null }>>` (partial because not every platform is guaranteed to appear in a given result set). Similarly, type `platformCounts` as `Partial<Record<PlatformId, number>>`. The `sourceToPlatform` function would need to return `PlatformId` (verified via `isPlatformId` from contracts) or keep returning `string` and let callers cast at the boundary.
- **Evidence**:

```typescript
// analytics.utils.ts:162-190
export function buildPlatformBreakdown({
  rows,
  responseTimes,
}: {
  rows: Array<{ source: string; count: number }>;
  responseTimes: Array<string | null>;
}): Record<string, { count: number; percentage: number; responseTime: string | null }> {
  const platformCounts: Record<string, number> = {};
  // ...
  const breakdown: Record<string, { count: number; percentage: number; responseTime: string | null }> = {};
```
