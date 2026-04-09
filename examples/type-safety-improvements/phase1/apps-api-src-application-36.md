# Audit: apps-api-src-application-36

**Files inspected**: 8
**Findings**: 9

## Summary

The batch contains two well-structured error modules (`training.errors.ts`, `triage.errors.ts`) with no issues, plus a clean pure-utils file (`training.utils.ts`). The main problems are clustered in three files: a double type cast in `token-usage.service.ts`, a locally-duplicated `StylePreferences` interface that already exists in `@slopweaver/contracts`, two `Record<string, ...>` usages that could be stricter, a manual `typeof result === "object"` guard that works around an already-typed SDK result, and a `(TRAINING_CATEGORIES as readonly string[])` cast that is the symptom of a design mismatch between the local category constant and the `buildCategoryStats` input type.

---

## Findings

### Finding 1: Double type cast when looking up model pricing

- **File**: `apps/api/src/application/token-usage/services/token-usage.service.ts:237`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `providerPricing?.[model as keyof typeof providerPricing]` narrows `model: string` to `keyof typeof providerPricing` with `as`, then the entire expression is immediately cast again to `{ input: number; output: number; cacheCreation?: number; cacheRead?: number } | undefined`. This double cast hides the fact that `MODEL_PRICING` already encodes all valid shapes — the second cast discards the more precise `const` type and replaces it with a hand-written approximation that is not kept in sync with the actual structure.
- **Suggestion**: Extract a helper `getModelPricing` that takes `provider` and `model` and returns a known union from `MODEL_PRICING` without any casts. A `Map` or a helper object indexed by `provider` → `model` can be statically typed so both casts become unnecessary. At a minimum, derive the return type from `(typeof MODEL_PRICING)[typeof provider][string]` rather than hand-writing it.
- **Evidence**:

```typescript
const pricing = providerPricing?.[model as keyof typeof providerPricing] as
  | { input: number; output: number; cacheCreation?: number; cacheRead?: number }
  | undefined;
```

---

### Finding 2: `TokenUsageStatsResponse` duplicates the contract schema shape

- **File**: `apps/api/src/application/token-usage/services/token-usage.service.ts:117-123`
- **Category**: sdk-type-duplication
- **Impact**: medium
- **Description**: `TokenUsageStatsResponse` is a hand-written interface whose shape exactly mirrors `tokenUsageStatsSchema` from `@slopweaver/contracts`. The comment even says "matches contract tokenUsageStatsSchema". If the contract shape changes, this local interface silently falls out of sync.
- **Suggestion**: Replace the local interface with `import type { tokenUsageStatsSchema } from "@slopweaver/contracts"; type TokenUsageStatsResponse = z.infer<typeof tokenUsageStatsSchema>;` — or import the already-exported `TokenUsageStats` type from contracts if one exists.
- **Evidence**:

```typescript
// Token usage stats response type (matches contract tokenUsageStatsSchema)
interface TokenUsageStatsResponse {
  byDay: { cost: string; date: string; tokens: number }[];
  byFeature: { cost: string; feature: string; tokens: number }[];
  byProvider: { cost: string; provider: string; tokens: number }[];
  totalCost: string;
  totalTokens: number;
}
```

---

### Finding 3: `StylePreferences` locally re-defined; already exported by `@slopweaver/contracts`

- **File**: `apps/api/src/application/training/services/training.service.ts:28-33`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `StylePreferences` is defined locally and exported from this service. An identical type — `StylePreferences = z.infer<typeof stylePreferencesSchema>` — is already exported from `@slopweaver/contracts` (see `packages/contracts/src/contracts/training/types.ts:24`). Keeping a local copy creates a divergence risk.
- **Suggestion**: Remove the local `export interface StylePreferences` and import the canonical type: `import type { StylePreferences } from "@slopweaver/contracts";`.
- **Evidence**:

```typescript
// training.service.ts
export interface StylePreferences {
  brevityLevel: number;
  formalityLevel: number;
  technicalLevel: number;
  toneLevel: number;
}
// contracts/training/types.ts
export type StylePreferences = z.infer<typeof stylePreferencesSchema>;
```

---

### Finding 4: `StylePreferenceLevels` in `training.utils.ts` is a third copy of the same shape

- **File**: `apps/api/src/application/training/utils/training.utils.ts:24-29`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: `StylePreferenceLevels` is a third definition of the same four-field `{ brevityLevel, formalityLevel, technicalLevel, toneLevel }` shape — alongside `StylePreferences` in `training.service.ts` and the contracts type. It is used as the parameter type of `estimateToneMatch` and `buildTrainingSystemPrompt`.
- **Suggestion**: Replace `StylePreferenceLevels` with the canonical `StylePreferences` type from `@slopweaver/contracts`, or at minimum with an import from `training.service.ts` once that is itself consolidated to the contracts type.
- **Evidence**:

```typescript
export interface StylePreferenceLevels {
  brevityLevel: number;
  formalityLevel: number;
  technicalLevel: number;
  toneLevel: number;
}
```

---

### Finding 5: Manual `typeof result === "object"` guard on an already-typed AI SDK result

- **File**: `apps/api/src/application/training/services/training-ai.service.ts:184-187`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `result` is the `Ok` value from `safeApiCall` wrapping `generateText()`. `generateText` returns `GenerateTextResult<M, T>` which already has a `usage` property typed by the Vercel AI SDK. The guard `typeof result === "object" && result !== null && "usage" in result` followed by `(result as { usage?: unknown }).usage` treats a fully-typed value as `unknown`, discards the SDK types, and casts back. This is a sign that `extractInputOutputTokensFromUsage` accepts `unknown` for `usage` — the guard here is defensive boilerplate that fights the type system rather than leveraging it.
- **Suggestion**: Access `result.usage` directly (it is typed on `GenerateTextResult`). If `extractInputOutputTokensFromUsage` requires `unknown`, fix its signature to accept `GenerateTextResult['usage']` instead. The guard and the cast can then be deleted.
- **Evidence**:

```typescript
const usage =
  typeof result === "object" && result !== null && "usage" in result
    ? (result as { usage?: unknown }).usage
    : undefined;
const { cacheCreationInputTokens, cacheReadInputTokens, inputTokens, outputTokens } = extractInputOutputTokensFromUsage(
  { usage },
);
```

---

### Finding 6: `Record<string, number | Date>` used for a Drizzle `set()` payload

- **File**: `apps/api/src/application/training/services/training.service.ts:235`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: `updateData: Record<string, number | Date>` is assembled by hand with string-keyed column names and then passed to `settingsTable.set()`. Drizzle's `set()` expects a typed `Partial<typeof settingsTable.$inferInsert>` — using `Record<string, ...>` bypasses that type check, meaning a typo in a column name (e.g., `"styleTonelevel"`) compiles silently.
- **Suggestion**: Build the update object as `Partial<typeof settingsTable.$inferInsert>` (or a narrowed picked type) and use the Drizzle column references directly rather than string keys:

```typescript
const updateData: Partial<typeof settingsTable.$inferInsert> = { updatedAt: new Date() };
if (data.toneLevel !== undefined) updateData.styleToneLevel = data.toneLevel;
```

- **Evidence**:

```typescript
const updateData: Record<string, number | Date> = { updatedAt: new Date() };
if (data.toneLevel !== undefined) {
  updateData["styleToneLevel"] = data.toneLevel;
}
```

---

### Finding 7: `Record<string, number>` for `categoryCounts` where a stricter type exists

- **File**: `apps/api/src/application/training/services/training.service.ts:306`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `categoryCounts: Record<string, number>` accumulates counts keyed by `example.category`. The category column comes from `trainingExamplesTable` and is constrained to the four values in `TRAINING_CATEGORIES`. Using `Record<string, number>` allows any arbitrary string key, hiding the domain constraint. The same `Record<string, number>` shape propagates to `buildCategoryStats` in `training.utils.ts`.
- **Suggestion**: Type as `Partial<Record<TrainingCategoryStat["category"], number>>` to enforce the domain constraint at compile time. This also removes the need for the `(TRAINING_CATEGORIES as readonly string[])` cast in `buildCategoryStats` (Finding 8).
- **Evidence**:

```typescript
const categoryCounts: Record<string, number> = {};
for (const example of examples) {
  const cat = example.category;
  categoryCounts[cat] = (categoryCounts[cat] ?? 0) + 1;
}
```

---

### Finding 8: `(TRAINING_CATEGORIES as readonly string[])` cast masks a solvable type mismatch

- **File**: `apps/api/src/application/training/utils/training.utils.ts:83`
- **Category**: type-cast
- **Impact**: low
- **Description**: `TRAINING_CATEGORIES` is `readonly ["tone", "brevity", "formality", "technical"]`. The cast to `readonly string[]` widens the element type to `string` so that `categoryCounts[category]` compiles with the `Record<string, number>` input type. This cast is the downstream symptom of `buildCategoryStats` accepting `Record<string, number>` instead of a tighter map. Fixing Finding 7 removes the need for this cast.
- **Suggestion**: Change `buildCategoryStats`'s `categoryCounts` parameter to `Partial<Record<TrainingCategoryStat["category"], number>>` and remove the cast. `TRAINING_CATEGORIES` elements are then already assignable as keys.
- **Evidence**:

```typescript
return (TRAINING_CATEGORIES as readonly string[])
  .filter((category) => (categoryCounts[category] ?? 0) > 0)
  .map((category) => ({
    category: category as TrainingCategoryStat["category"],
    ...
  }));
```

---

### Finding 9: `Record<string, typeof examples>` in `buildTrainingSystemPrompt` for a category-keyed map

- **File**: `apps/api/src/application/training/services/training-ai.utils.ts:66`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `byCategory: Record<string, typeof examples>` groups examples by category string. Like `categoryCounts` above, the key is effectively `TrainingCategoryStat["category"]` (the four known values). Using `string` as the key hides the domain and allows accessing with arbitrary strings without a type error.
- **Suggestion**: Type as `Partial<Record<TrainingCategoryStat["category"], typeof examples>>` and import `TrainingCategoryStat` from `training.utils.ts`. This makes the constraint explicit without requiring a cast on lookup.
- **Evidence**:

```typescript
const byCategory: Record<string, typeof examples> = {};
for (const ex of examples) {
  byCategory[ex.category] ??= [];
  ...
}
```
