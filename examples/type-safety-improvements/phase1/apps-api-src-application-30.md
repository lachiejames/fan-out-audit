# Audit: apps-api-src-application-30

**Files inspected**: 8
**Findings**: 4

## Summary

These files cover push-notification rate limiting, notification routing utilities, saved-filter CRUD, AI model policy enforcement, settings service, suggested-actions error types, and a suggestion feedback repository. The code is generally well-typed; findings centre on three weak spots: a `Record<string, unknown>` accumulator used as a Drizzle update payload, an untyped `string` where a literal union already exists in the DB schema, and two `unknown` parameters in the feedback repository that have richer contract types available.

## Findings

### Finding 1: `Record<string, unknown>` update accumulator loses Drizzle column types

- **File**: `apps/api/src/application/settings/services/settings.service.ts:215`
- **Category**: record-weakening
- **Impact**: high
- **Description**: `drizzleData` is typed as `Record<string, unknown>` so that `updatedAt` and `voicePrivacyConsentAt` can be injected. This bypasses Drizzle's column-level type checking on `.set()`: any misspelled key or wrong value type will compile silently. Because `settingsTable.$inferInsert` already expresses the exact allowed shape, the cast-via-spread pattern should be replaced with a properly typed partial object.
- **Suggestion**: Build a `Partial<typeof settingsTable.$inferInsert>` value directly. Cast only the one field that needs transformation (`voicePrivacyConsentAt`) inside a typed assignment, removing the `Record<string, unknown>` entirely.
- **Evidence**:

```typescript
// settings.service.ts:215-218
const drizzleData: Record<string, unknown> = { ...data, updatedAt: new Date() };
if (typeof drizzleData["voicePrivacyConsentAt"] === "string") {
  drizzleData["voicePrivacyConsentAt"] = new Date(drizzleData["voicePrivacyConsentAt"]);
}
```

---

### Finding 2: `dimensionType` typed as `string` instead of a literal union

- **File**: `apps/api/src/application/suggested-actions/repositories/suggestion-feedback.repository.ts:224`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: The `upsertDailyAggregateCounts` parameter `dimensionType` is typed as `string`, but the DB column comment documents only three valid values: `"overall" | "action_type" | "intent_category"`. The table column is a plain `text`, so no DB constraint prevents invalid values. Callers can silently pass any string, inserting rows with unexpected dimension types that will never be queried. A local literal union type (or a shared enum) would make invalid values a compile-time error.
- **Suggestion**: Replace `dimensionType: string` with `dimensionType: "overall" | "action_type" | "intent_category"` (or export a named `DimensionType` from the table file and import it here).
- **Evidence**:

```typescript
// suggestion-feedback.repository.ts:218-225
async upsertDailyAggregateCounts({
  userId,
  bucketDate,
  dimensionType,      // <-- typed as plain string
  dimensionKey,
  field,
}: {
  userId: string;
  bucketDate: Date;
  dimensionType: string;   // should be "overall" | "action_type" | "intent_category"
```

---

### Finding 3: `suggestedAction` and `modifications` params typed as `unknown` in `insertFeedback`

- **File**: `apps/api/src/application/suggested-actions/repositories/suggestion-feedback.repository.ts:178-179`
- **Category**: any-usage
- **Impact**: medium
- **Description**: `suggestedAction: unknown` and `modifications: unknown` are passed straight into `db.insert()`. The contract package already exports `CompoundSuggestion` (imported elsewhere in the same file for `createPresentation`). Using `unknown` means callers can pass anything without a compile-time error, and the DB layer silently serialises it to JSONB. `suggestedAction` should be `CompoundSuggestion`; `modifications` would at minimum benefit from `Record<string, unknown> | null`, but ideally a typed modifications interface from the contracts.
- **Suggestion**: Change `suggestedAction: unknown` to `suggestedAction: CompoundSuggestion` (already imported). Introduce or reuse a contract type for `modifications`, or type it as `Record<string, unknown> | null` until a richer type is defined.
- **Evidence**:

```typescript
// suggestion-feedback.repository.ts:11 — CompoundSuggestion already imported
import type { CompoundSuggestion } from "@slopweaver/contracts";

// suggestion-feedback.repository.ts:178-179 — but not used here
    suggestedAction: unknown,
    ...
    modifications: unknown,
```

---

### Finding 4: Non-null assertion `row!.id` hides potential undefined from `returning()`

- **File**: `apps/api/src/application/suggested-actions/repositories/suggestion-feedback.repository.ts:64` and `:203`
- **Category**: type-cast
- **Impact**: low
- **Description**: After `.returning({ id: ... })`, `row` is destructured as potentially `undefined` but immediately accessed with `row!.id` — a non-null assertion that suppresses TypeScript's undefined check. If the insert were to return an empty array (e.g., a policy veto, race condition, or future conditional insert), the runtime would throw an opaque `TypeError: Cannot read properties of undefined`. The pattern used in `saved-filters.service.ts` (check `!row` and throw a named error) is safer.
- **Suggestion**: Replace `return row!.id` with an explicit guard:
  ```typescript
  if (!row) throw new Error("insert returned no row");
  return row.id;
  ```
- **Evidence**:

```typescript
// suggestion-feedback.repository.ts:62-65
      .returning({ id: suggestionPresentationsTable.id });

    return row!.id;   // non-null assertion

// suggestion-feedback.repository.ts:201-203
      .returning({ id: suggestionFeedbackTable.id });

    return row!.id;   // non-null assertion
```
