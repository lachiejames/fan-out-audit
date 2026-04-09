# Audit: apps-api-src-application-37

**Files inspected**: 8
**Findings**: 7

## Summary

The triage services batch is generally well-typed, using `neverthrow` Result types, Drizzle-inferred table types, and `satisfies` guards consistently. The main opportunities cluster around:

1. `string` used for triage action and rule-type fields where the contracts package already exports stricter union types (`TriageAction`, `LearnedRuleType`).
2. The `LearnedRule` interface in `triage-learning.service.ts` duplicates the `LearnedRule` type already exported from `@slopweaver/contracts`.
3. Two inline anonymous return-type shapes in `triage-learning.service.ts` duplicate the contract-exported `LearningProgressCandidate` and `OverrideStatsResponse` types.
4. An empty `catch {}` block in `triage-rules.service.ts` that silently swallows errors.
5. A `sql<number>` aggregation in `triage-session.service.ts` that carries an eslint-disable comment acknowledging a known runtime/type mismatch (Drizzle returns string at runtime despite `sql<number>`).

---

## Findings

### Finding 1: `LearnedRule` interface duplicates contract-exported type

- **File**: `apps/api/src/application/triage/services/triage-learning.service.ts:28-34`
- **Category**: duplicate-type
- **Impact**: high
- **Description**: `LearnedRule` is defined locally as an interface with `type: "sender_domain" | "sender_email" | "platform"`, `action: string`, `confidence: number`, `occurrences: number`, `pattern: string`. The contracts package exports an identical `LearnedRule` type (inferred from `learnedRuleSchema` at `packages/contracts/src/contracts/triage/schemas.ts:347`) with the same shape. The service even re-exports the local interface so consumers import from `triage-learning.service.ts` rather than the canonical source.
- **Suggestion**: Delete the local `LearnedRule` interface and import `LearnedRule` from `@slopweaver/contracts`. Update all import sites (there is at least one in `triage-classification.service.ts:32`).
- **Evidence**:

```typescript
// triage-learning.service.ts:28-34 — duplicate definition
export interface LearnedRule {
  type: "sender_domain" | "sender_email" | "platform";
  pattern: string;
  action: string;
  confidence: number;
  occurrences: number;
}

// packages/contracts/src/contracts/triage/schemas.ts:340-347 — canonical source
export const learnedRuleSchema = z.object({
  action: z.string(),
  confidence: z.number().min(0).max(1),
  occurrences: z.number().int().min(0),
  pattern: z.string(),
  type: learnedRuleTypeSchema,
});
export type LearnedRule = z.infer<typeof learnedRuleSchema>;
```

---

### Finding 2: `getLearningProgress` return type duplicates `LearningProgressCandidate` contract type

- **File**: `apps/api/src/application/triage/services/triage-learning.service.ts:225-242`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: The return type of `getLearningProgress` is a verbatim inline anonymous array matching the `LearningProgressCandidate` type exported by `@slopweaver/contracts` (`packages/contracts/src/contracts/triage/schemas.ts:368-376`). The local `candidates` array also carries an identical inline type annotation. This creates two places to update if the schema changes.
- **Suggestion**: Import `LearningProgressCandidate` from `@slopweaver/contracts` and use it as the return type and element type for the `candidates` array.
- **Evidence**:

```typescript
// triage-learning.service.ts:225-242 — inline shape, repeated twice
async getLearningProgress({ userId }: { userId: string }): Promise<
  {
    action: string;
    currentCount: number;
    pattern: string;
    remaining: number;
    threshold: number;
    type: "sender_domain" | "sender_email";
  }[]
>
// ...
const candidates: {
  action: string;
  currentCount: number;
  pattern: string;
  remaining: number;
  threshold: number;
  type: "sender_domain" | "sender_email";
}[] = [];
```

---

### Finding 3: `getOverrideStats` return type duplicates `OverrideStatsResponse` contract type

- **File**: `apps/api/src/application/triage/services/triage-learning.service.ts:381-395`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: The return type of `getOverrideStats` is declared as an inline object shape that exactly mirrors the `OverrideStatsResponse` type already exported from `@slopweaver/contracts` (`packages/contracts/src/contracts/triage/schemas.ts:352-363`).
- **Suggestion**: Import `OverrideStatsResponse` from `@slopweaver/contracts` and use it as the explicit return type.
- **Evidence**:

```typescript
// triage-learning.service.ts:389-394 — inline duplicate
}: Promise<{
  totalOverrides: number;
  learnedRulesCount: number;
  topOverriddenActions: { aiSuggestion: string; userAction: string; count: number }[];
}>
```

---

### Finding 4: `RecordOverrideParams.aiSuggestion` and `userAction` typed as `string` instead of `TriageAction`

- **File**: `apps/api/src/application/triage/services/triage-learning.service.ts:19-26`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `RecordOverrideParams` declares `aiSuggestion: string` and `userAction: string`. Both fields represent triage action values (`"archive"`, `"todo"`, `"skip"`, `"defer"`, `"reply"`), and the contracts package exports `TriageAction` as a precise union for exactly these values. Using `string` allows any string to be passed at the call site, bypassing compile-time validation. The same issue applies to `normalizeSuggestionAction`'s parameter and return type, and to `recordSuggestionFeedback`'s `suggestedAction` / `actualAction` params.
- **Suggestion**: Type `aiSuggestion`, `userAction`, `suggestedAction`, `actualAction`, and the return of `normalizeSuggestionAction` as `TriageAction` (or `string` → `TriageAction` where coercion happens). For `normalizeSuggestionAction`, the input may intentionally be a wider `string` coming from a suggestion-action vocabulary — if so, `string` input with `TriageAction` return (and exhaustive handling) is appropriate.
- **Evidence**:

```typescript
// triage-learning.service.ts:19-26
export interface RecordOverrideParams {
  userId: string;
  sessionId: string;
  contentId: string;
  aiSuggestion: string; // should be TriageAction
  userAction: string; // should be TriageAction
  aiConfidence: number | null;
}
```

---

### Finding 5: `LearnedRule.action` typed as `string` instead of `TriageAction`

- **File**: `apps/api/src/application/triage/services/triage-learning.service.ts:31`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: The local `LearnedRule` interface (and the corresponding `learnedRuleSchema` in contracts) uses `action: string`. When rules are consumed in `triage-classification.service.ts:247` (`rule.action !== classification.suggestion`), the comparison is against a typed `TriageAction`. Narrowing `action` to `TriageAction` on the rule type would catch mismatches at compile time. (This finding is secondary to Finding 1 — fixing Finding 1 by importing from contracts will also pull in this issue from the canonical schema; both should be tightened together.)
- **Suggestion**: Change `action: string` to `action: TriageAction` in both the local interface and `learnedRuleSchema`.
- **Evidence**:

```typescript
// triage-learning.service.ts:31
action: string;

// triage-classification.service.ts:247 — comparison site
if (rule.action !== classification.suggestion) { // classification.suggestion is TriageAction
```

---

### Finding 6: Silent `catch {}` in `triage-rules.service.ts`

- **File**: `apps/api/src/application/triage/services/triage-rules.service.ts:72`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The rule detection loop has an empty `catch {}` block. When `rule.detect(item)` throws, the exception is swallowed entirely — no logging, no error propagation, and the item is simply treated as unmatched (falls through to `needsReview`). This hides bugs in rule implementations and makes debugging silent failures impossible.
- **Suggestion**: At minimum, log the error with context (`ruleId`, `contentId`). Ideally also annotate what kinds of errors are expected vs unexpected so future maintainers understand the intent.
- **Evidence**:

```typescript
// triage-rules.service.ts:55-73
for (const rule of platformRules) {
  try {
    if (rule.detect(item)) {
      // ...
    }
  } catch {} // swallows any error from rule.detect — no logging
}
```

---

### Finding 7: `sql<number>` aggregation result requires runtime coercion with suppressed lint warning

- **File**: `apps/api/src/application/triage/services/triage-session.service.ts:782`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `sql<number>\`COALESCE(SUM(...))\``is annotated as`sql<number>`but the code immediately wraps it with`Number(...)`and carries an`// eslint-disable-next-line @typescript-eslint/no-unnecessary-type-conversion`comment acknowledging that Drizzle returns a string at runtime despite the`<number>`generic. This pattern — a deliberate lie in the type annotation — means TypeScript will silently allow treating the raw value as`number`elsewhere without the`Number()` wrapper.
- **Suggestion**: Type the column as `sql<string>` to match actual runtime behavior, then convert with `Number()` explicitly. This makes the coercion visible to readers without needing a lint suppression comment, and prevents callers from accidentally skipping the conversion step.
- **Evidence**:

```typescript
// triage-session.service.ts:765-782
.select({
  totalCostMicros: sql<number>`COALESCE(SUM(${llmCallsTable.totalCostMicros}), 0)`,
})
// ...
// eslint-disable-next-line @typescript-eslint/no-unnecessary-type-conversion -- Drizzle sql<number> returns string at runtime
const totalCostMicros = Number(dbResult.value[0]?.totalCostMicros ?? 0);
```
