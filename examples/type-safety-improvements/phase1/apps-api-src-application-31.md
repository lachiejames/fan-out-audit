# Audit: apps-api-src-application-31

**Files inspected**: 8
**Findings**: 5

## Summary

These files implement the suggestion and suggested-actions subsystems: feedback classification, feedback persistence, compound suggestion building, step building, and the legacy suggestions list service. The main type-safety issues are (1) repeated unsafe `as CompoundSuggestion` casts required because `SuggestionPresentation` stores JSONB columns as `unknown`, (2) a `Record<SuggestionFeedbackType, string>` map whose value type is `string` instead of the narrower field-name union, causing a downstream `as "acceptedCount"` cast at every call site, (3) `StoredPrediction.predictedActions` and `.contentSignals` typed as `unknown` forcing casts during normalization, (4) `getStoredDetailsForAction` returning `Record<string, unknown>` when the prediction item's `details` is already typed, and (5) a `Record<string, number>` `byPlatform` map built from freeform DB strings then immediately destructured with hardcoded platform keys.

## Findings

### Finding 1: `FEEDBACK_TYPE_TO_FIELD` value typed as `string` forces repeated `as "acceptedCount"` casts

- **File**: `apps/api/src/application/suggested-actions/services/suggestion-feedback.service.ts:33`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `FEEDBACK_TYPE_TO_FIELD` maps `SuggestionFeedbackType` → `string`, but the repository's `upsertDailyAggregateCounts` expects the much narrower union `"shownCount" | "acceptedCount" | "modifiedCount" | "rejectedCount" | "ignoredCount" | "dismissedCount"`. Because the map's value is `string`, every call site must cast `field as "acceptedCount"` (lines 653, 663, 674), which silently accepts any string and defeats exhaustiveness checking.
- **Suggestion**: Tighten the map's value type to the exact field-name union used by the repository parameter. A mapped type `{ [K in SuggestionFeedbackType]: AggregateField }` would also guarantee every feedback type has a mapping and remove all three casts.
- **Evidence**:

```typescript
// service.ts:33
const FEEDBACK_TYPE_TO_FIELD: Record<SuggestionFeedbackType, string> = {
  accepted: "acceptedCount",
  dismissed: "dismissedCount",
  ...
};

// service.ts:643-653
const field = FEEDBACK_TYPE_TO_FIELD[classified.feedbackType];
if (!field) return;
await this.repo.upsertDailyAggregateCounts({
  ...
  field: field as "acceptedCount",   // cast needed because value is string
});
```

Fix:

```typescript
type AggregateField = "shownCount" | "acceptedCount" | "modifiedCount" | "rejectedCount" | "ignoredCount" | "dismissedCount";
const FEEDBACK_TYPE_TO_FIELD: Record<SuggestionFeedbackType, AggregateField> = { ... };
// field is now AggregateField — no cast needed
```

---

### Finding 2: Repeated `as CompoundSuggestion` casts on JSONB columns with no narrowing

- **File**: `apps/api/src/application/suggested-actions/services/suggestion-feedback.service.ts:149`
- **Category**: type-cast
- **Impact**: high
- **Description**: `SuggestionPresentation.primarySuggestion` and `.alternativeSuggestions` are Drizzle JSONB columns typed as `unknown`. They are cast to `CompoundSuggestion` / `CompoundSuggestion[]` at least 7 times throughout the service (lines 149, 270, 317, 419, 732, 733) without any runtime validation. If data stored in the DB drifts from the schema (e.g. after a contract change) these casts silently produce malformed objects that fail later, with no error surface at the point of read.
- **Suggestion**: Add a Zod-based type guard at the repository layer (in `SuggestionFeedbackRepository.findLatestPendingPresentation` / `findLatestPresentation`) that parses `primarySuggestion` and `alternativeSuggestions` using the `compoundSuggestionSchema` from `@slopweaver/contracts`. Return typed values so the service never needs to cast. Alternatively, define the Drizzle column with a custom type using `.$type<CompoundSuggestion>()` if data integrity at insert time is already guaranteed.
- **Evidence**:

```typescript
// Lines 149, 270, 317, 419 — same unsafe pattern repeated
const primarySuggestion = presentation.primarySuggestion as CompoundSuggestion;

// Line 732-733
return [
  presentation.primarySuggestion as CompoundSuggestion,
  ...((presentation.alternativeSuggestions as CompoundSuggestion[]) ?? []),
];
```

---

### Finding 3: `StoredPrediction.predictedActions` and `.contentSignals` typed as `unknown`, forcing inline casts

- **File**: `apps/api/src/application/suggested-actions/utils/suggested-actions-builder.ts:62`
- **Category**: any-usage
- **Impact**: medium
- **Description**: The `StoredPrediction` interface (lines 60-71) types `predictedActions` and `contentSignals` as `unknown`. Inside `normalizePredictionRow`, both are immediately cast with `as Array<{...}>` and `as PredictionContentSignals | undefined` respectively, with a comment acknowledging the type mismatch. The `PredictionOutput` object assembled from these fields is then cast `as PredictionOutput` (line 102). Three unsafe casts stem from this one interface gap.
- **Suggestion**: Type `predictedActions` as `Array<{ action: string; details: Record<string, unknown>; order: number }> | null` and `contentSignals` as `PredictionContentSignals | null` in the `StoredPrediction` interface. These are the shapes already stored and immediately asserted — making the interface honest eliminates all three casts and the `as PredictionOutput` assembly cast.
- **Evidence**:

```typescript
// suggested-actions-builder.ts:62-70
export interface StoredPrediction {
  predictedAction: string;
  predictedActions: unknown;          // ← typed unknown
  contentSignals: unknown;            // ← typed unknown
  ...
}

// Lines 92-102 — forced by unknown types above
contentSignals: prediction.contentSignals as PredictionContentSignals | undefined,
predictedActions: prediction.predictedActions as
  | Array<{ action: string; details: Record<string, unknown>; order: number }>
  | undefined,
} as PredictionOutput;
```

---

### Finding 4: `buildTypedDetails` returns `Record<string, unknown>` instead of the contract's discriminated union

- **File**: `apps/api/src/application/suggested-actions/utils/suggested-actions-steps.utils.ts:83`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `buildTypedDetails` and `buildStepLabel` both accept/return `Record<string, unknown>` for the details object, and the `SuggestionStep` type in `@slopweaver/contracts` uses a discriminated union of action-specific detail shapes. The return type is therefore wider than what the contract demands, so the caller in `buildSuggestionSteps` must still do `} as SuggestionStep` on line 65 to satisfy the contract type. This also means `buildStepLabel` accesses `details["taskTitle"]` etc. via index without narrowing, relying on `String(title)` coercions instead of typed access.
- **Suggestion**: Import the discriminated `SuggestionStepDetails` union (or per-action detail types) from `@slopweaver/contracts` and type the return of `buildTypedDetails` as that union. This removes the `as SuggestionStep` cast on line 65 and lets `buildStepLabel` receive a typed discriminated union it can narrow with a `switch`.
- **Evidence**:

```typescript
// suggested-actions-steps.utils.ts:73-83
export function buildTypedDetails({...}: {
  action: SuggestedActionType;
  raw: Record<string, unknown>;
  ...
}): Record<string, unknown> {   // ← should be the discriminated union

// Line 59-65
return {
  action: actionType,
  details,
  ...
} as SuggestionStep;            // ← cast needed because details is Record<string,unknown>
```

---

### Finding 5: `SUGGESTION_PREFIX_MAP` keyed by `string` instead of a const-keyed type; `byPlatform` built as `Record<string, number>` then accessed with hardcoded keys

- **File**: `apps/api/src/application/suggestions/utils/suggestions.utils.ts:184`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `SUGGESTION_PREFIX_MAP` is typed `Record<string, SuggestionType>` even though its keys are five specific prefix literals. A `const` assertion is already present on the value arrays elsewhere in the file, but not here — so the TypeScript inference loses the key set. Separately, in `suggestions.service.ts` lines 566-597, `byPlatform` is typed as `Record<string, number>`, built from raw DB `source` strings, then immediately destructured with three hardcoded keys `"google-gmail"`, `"linear"`, `"slack"` (line 158-160). Since the contract schema uses `z.record(z.string(), ...)` the broader type is intentional, but the construction/consumption mismatch makes it easy for platform keys to go stale silently.
- **Suggestion**: For `SUGGESTION_PREFIX_MAP`, use a `const` assertion or a mapped type over the known prefix literals (`"urgent-" | "todo-" | "awaiting-" | "deadline-" | "cross-"`) as keys — this makes `Object.entries` iteration exhaustive and catches typos. For `byPlatform`, the hardcoded access on lines 158-160 in the service is the real fragility: consider deriving `relatedItems` directly from the `byPlatform` map (e.g., `Object.fromEntries(...)`) rather than selecting three hardcoded keys.
- **Evidence**:

```typescript
// suggestions.utils.ts:184
export const SUGGESTION_PREFIX_MAP: Record<string, SuggestionType> = {
  "awaiting-": "awaiting_response",
  "cross-": "cross_platform",
  ...
};

// suggestions.service.ts:158-160
relatedItems: {
  "google-gmail": topic.byPlatform["google-gmail"] ?? null,
  linear: topic.byPlatform["linear"] ?? null,
  slack: topic.byPlatform["slack"] ?? null,
},
```
