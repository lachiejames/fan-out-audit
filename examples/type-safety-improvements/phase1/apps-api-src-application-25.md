# Audit: apps-api-src-application-25

**Files inspected**: 8
**Findings**: 5

## Summary

These files implement the prediction engine (complexity assessment, memory formatting, prediction parsing, thinking budget calculation, token estimation) and the priority-scoring subsystem (error types, scoring service, VIP contacts service). The code is generally well-typed, but there are a few places where loose types leak in: a hand-rolled `NormalizedPredictionOutput` that duplicates contract types using bare `string` instead of the narrower union literals, redundant intermediate variable assignments that widen Anthropic SDK block types unnecessarily, a `Record<string, unknown>` context bag on the error type where a mapped type would be safer, and an unsafe `metadata` parameter typed as `Record<string, unknown> | null` whose fields are accessed with string-key indexing rather than a known shape.

---

## Findings

### Finding 1: `NormalizedPredictionOutput` duplicates contract types with wider `string` fields

- **File**: `apps/api/src/application/prediction/utils/prediction-parser.ts:92`
- **Category**: duplicate-type
- **Impact**: high
- **Description**: `NormalizedPredictionOutput` is a hand-rolled interface that re-declares fields already fully typed in `@slopweaver/contracts`. Critically, `predictedAction`, `predictedTiming`, and the `action` field inside `predictedActions` are typed as bare `string` rather than the narrower union literals exported by the contracts package (`PredictedAction`, `PredictedTiming`, `PredictionCompoundAction`). This means callers of `normalizePredictionOutput` (or any function returning this type) lose exhaustiveness checking when switching on these fields. The contracts already export `PredictedAction = z.infer<typeof predictedActionSchema>` (a string union of 10 values) and `PredictionCompoundAction` which includes the same action union.
- **Suggestion**: Replace the hand-rolled interface with types composed from the contracts package:

  ```typescript
  import type {
    PredictedAction,
    PredictedTiming,
    PredictionCompoundAction,
    PredictionContentSignals,
    PredictionIntentCategory,
  } from "@slopweaver/contracts";

  export interface NormalizedPredictionOutput {
    actionExplanation: string;
    confidence: number;
    contentSignals: PredictionContentSignals;
    intentCategory: PredictionIntentCategory;
    predictedAction: PredictedAction; // was: string
    predictedActions: PredictionCompoundAction[]; // was: Array<{ action: string; details: Record<string, unknown>; order: number }>
    predictedResponse: string | null;
    predictedTiming: PredictedTiming | null; // was: string | null
    predictedTone: number | null;
    reasoning: string;
  }
  ```

- **Evidence**:

```typescript
export interface NormalizedPredictionOutput {
  actionExplanation: string;
  confidence: number;
  contentSignals: PredictionContentSignals;
  intentCategory: PredictionIntentCategory;
  predictedAction: string; // <-- should be PredictedAction (union of 10 literals)
  predictedActions: Array<{ action: string; details: Record<string, unknown>; order: number }>;
  predictedResponse: string | null;
  predictedTiming: string | null; // <-- should be PredictedTiming | null
  predictedTone: number | null;
  reasoning: string;
}
```

---

### Finding 2: Redundant intermediate typed variables widen SDK `ContentBlock` subtypes

- **File**: `apps/api/src/application/prediction/utils/prediction-parser.ts:121`
- **Category**: type-cast
- **Impact**: low
- **Description**: Inside `extractBlocks`, after narrowing a `ContentBlock` with `block.type === "thinking"`, the code reassigns the already-narrowed `block` to a new `const thinkingBlock: ThinkingBlock` variable, and similarly for `textBlock: TextBlock`. These explicit type annotations are redundant — TypeScript has already narrowed `block` to `ThinkingBlock` / `TextBlock` via the discriminant — but they constitute an implicit widening assertion that could hide a future change if the SDK's discriminant ever shifts. More importantly they import `ThinkingBlock` and `TextBlock` directly from the SDK resource path (`@anthropic-ai/sdk/resources/messages`), which is a deep import that is subject to breakage on minor SDK updates.
- **Suggestion**: Remove the intermediate variable and use the already-narrowed `block` directly. If a local alias is needed for readability, `const thinkingBlock = block` (no annotation) is sufficient. The `import type { ContentBlock, TextBlock, ThinkingBlock }` line can be reduced to just `ContentBlock`.
- **Evidence**:

```typescript
import type { ContentBlock, TextBlock, ThinkingBlock } from "@anthropic-ai/sdk/resources/messages";
// ...
for (const block of content) {
  if (block.type === "thinking") {
    const thinkingBlock: ThinkingBlock = block; // redundant — block is already ThinkingBlock
    thinkingSummary = thinkingBlock.thinking.slice(0, 500);
    thinkingTokens = Math.ceil(thinkingBlock.thinking.length / 4);
  } else if (block.type === "text") {
    const textBlock: TextBlock = block; // redundant — block is already TextBlock
    textContent += textBlock.text;
  }
}
```

---

### Finding 3: `Record<string, unknown>` context on `CALCULATION_ERROR` loses key safety

- **File**: `apps/api/src/application/priority-scoring/errors/priority-scoring.errors.ts:19`
- **Category**: record-weakening
- **Impact**: low
- **Description**: The `CALCULATION_ERROR` variant carries `context?: Record<string, unknown>`, an open-ended bag. Every call site in the codebase will pass arbitrary keys with no compile-time contract. Because the error discriminant is `"CALCULATION_ERROR"`, callers that want to log or serialize the context can do so — but they get no type help on what keys to expect. A minimal mapped type (or an inline object type at the call sites) would constrain what context can be attached.
- **Suggestion**: If the only current call sites pass a known shape (e.g., `{ contentId: string }`), tighten the type at the error definition. If context is genuinely open-ended, document it with a type alias so it can be tightened later:
  ```typescript
  type CalculationErrorContext = Record<string, unknown>; // tighten when callers stabilize
  | { type: "CALCULATION_ERROR"; message: string; context?: CalculationErrorContext }
  ```
  Alternatively, if specific keys are always present, use an intersection: `{ contentId?: string } & Record<string, unknown>`.
- **Evidence**:

```typescript
| { type: "CALCULATION_ERROR"; message: string; context?: Record<string, unknown> };

export const PriorityScoringErrors = {
  calculation: (message: string, context?: Record<string, unknown>): PriorityScoringError => ({
    message,
    type: "CALCULATION_ERROR",
    ...(context !== undefined && { context }),
  }),
```

---

### Finding 4: `metadata` typed as `Record<string, unknown> | null` with unsafe string-key access

- **File**: `apps/api/src/application/priority-scoring/services/priority-scoring.service.ts:427`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `calculateMessageType` accepts `metadata: Record<string, unknown> | null` and then accesses it with the string keys `"user"` and `"channel"` directly: `"user" in metadata && metadata["user"] !== undefined` and `"channel" in metadata`. The intent is clearly Slack-specific metadata, but there is no type to describe its shape — the service has to use `in` checks and index access because TypeScript does not know what keys are valid. This is unsafe `unknown` extraction via an overly wide record type: if the key names ever change in the Slack payload (or a non-Slack platform passes metadata with `"user"` and `"channel"` for different reasons), the scoring silently misfires.
- **Suggestion**: Define a minimal discriminated shape for known metadata variants, or narrow inside the function after a platform guard. At minimum, introduce a typed interface for the Slack metadata subset:
  ```typescript
  interface SlackMessageMetadata {
    user?: string;
    channel?: string;
    thread_ts?: string;
    [key: string]: unknown;
  }
  ```
  Then accept `metadata: SlackMessageMetadata | Record<string, unknown> | null` or add a platform parameter and narrow before accessing keys.
- **Evidence**:

```typescript
private calculateMessageType({
  content,
  metadata,
}: {
  content: string;
  metadata: Record<string, unknown> | null;  // <-- open bag, Slack keys accessed below
}): number {
  // ...
  const hasMentions = "user" in metadata && metadata["user"] !== undefined;
  const isSlackThreadStarter = "channel" in metadata;
```

---

### Finding 5: `compoundActionOutputSchema` in `prediction-parser.ts` duplicates `predictionCompoundActionSchema` from contracts

- **File**: `apps/api/src/application/prediction/utils/prediction-parser.ts:21`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `compoundActionOutputSchema` defines an action enum with all 10 values and a `details: z.record(z.string(), z.unknown()).default({})` field. This is structurally identical to `predictionCompoundActionSchema` in `packages/contracts/src/contracts/predictions/schemas.ts` (line 43). The two schemas must be kept in sync manually — any divergence (e.g., adding an 11th action to contracts but missing it here) would cause silent parse failures at runtime where the parser rejects valid Claude responses. Similarly, `predictionOutputSchema`'s `predictedAction` enum and `intentCategory` enum duplicate `predictedActionSchema` and `predictionIntentCategorySchema` from contracts.
- **Suggestion**: Import and reuse the schemas from the contracts package rather than redefining them. The `predictionOutputSchema` in the parser is the parser-internal validation schema and may intentionally be a strict subset, but in that case the duplication should be explicitly documented. At minimum, the `action` enum inside `compoundActionOutputSchema` and the `predictedAction` enum in `predictionOutputSchema` should be replaced with imports:
  ```typescript
  import {
    predictedActionSchema,
    predictionIntentCategorySchema,
    predictionCompoundActionSchema,
  } from "@slopweaver/contracts";
  // or from the internal schemas path if circular imports are a concern
  ```
- **Evidence**:

```typescript
// prediction-parser.ts (apps/api)
const compoundActionOutputSchema = z.object({
  action: z.enum([
    "reply",
    "quick_reply",
    "create_task",
    "schedule",
    "review",
    "forward",
    "archive",
    "snooze",
    "save_resource",
    "ignore",
  ]),
  details: z.record(z.string(), z.unknown()).default({}),
  order: z.number().int().min(1).max(4),
});

// packages/contracts/src/contracts/predictions/schemas.ts (identical shape)
export const predictionCompoundActionSchema = z.object({
  action: predictedActionSchema, // same 10-value enum
  details: z.record(z.string(), z.unknown()).default({}),
  order: z.number().int().min(1).max(4),
});
```
