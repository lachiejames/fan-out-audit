# Audit: apps-api-src-shared-5

**Files inspected**: 8
**Findings**: 5

## Summary

This slice covers several core table files. `contents.table.ts` has a `metadata` column typed only as `Record<string, unknown>` and a `priorityFactors` jsonb column that is typed with an inline anonymous type — good practice but isolated to this one usage. `settings.table.ts` has several inline jsonb column types that duplicate shape definitions from other layers. `enums.ts` is clean with no issues. The `feature-flags.table.ts` uses a plain `text` column for `featureId` where a union of known feature IDs would be safer.

## Findings

### Finding 1: `metadata` column in `contents.table.ts` typed as `Record<string, unknown>`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/contents.table.ts:59`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: The `metadata` jsonb column is typed as `Record<string, unknown>`. Given that `platformIdentifiers` was specifically broken out into its own typed jsonb column (line 66), the remaining `metadata` column presumably holds platform-specific data that is looser. However, the type declaration provides no guidance on what keys to expect. Downstream code reading `content.metadata` will need to assert types for every access.
- **Suggestion**: Define a `ContentMetadata` interface with known optional keys (sender details, message flags, etc.) to at least document the shape. Even a partial type is better than the fully open `Record<string, unknown>`.
- **Evidence**:

```typescript
metadata: jsonb("metadata").$type<Record<string, unknown>>().default({}).notNull(),
```

### Finding 2: `styleProfile` in `settings.table.ts` has inline anonymous jsonb type with nested `Record<string, ...>` for calibration data

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/settings.table.ts:115-155`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: The `styleProfile` jsonb column is typed with a long inline anonymous type containing nested `Record<string, { accuracy: number; accepted: number; shown: number }>` for `byActionType`, `byIntentCategory`, and `bySenderDomain`. These are all structurally identical. The keys of `byActionType` and `byIntentCategory` correspond to enum values (`WorkItemType`, intent categories) but are typed as plain `string`, preventing type-safe lookups.
- **Suggestion**: Extract the repeated `{ accuracy: number; accepted: number; shown: number }` shape into a named `CalibrationBucket` interface. For `byActionType`, use `Partial<Record<WorkItemType, CalibrationBucket>>` to leverage the existing `WorkItemType` enum. Export the full `StyleProfile` interface from the table file or from a shared types file.
- **Evidence**:

```typescript
byActionType: Record<string, { accuracy: number; accepted: number; shown: number }>;
byIntentCategory: Record<string, { accuracy: number; accepted: number; shown: number }>;
bySenderDomain: Record<string, { accuracy: number; accepted: number; shown: number }>;
```

### Finding 3: `notificationPreferences` in `settings.table.ts` uses inline anonymous jsonb type with hardcoded string keys

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/settings.table.ts:79-92`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The `notificationPreferences` jsonb column is typed with an inline anonymous object with keys `urgent_mentions`, `todo_suggestions`, `daily_digest`, `quiet_hours`. These keys are hardcoded in the DB definition but likely match a type from `@slopweaver/contracts`. Defining this inline in the table schema creates a potential for drift.
- **Suggestion**: Extract this type to a named `NotificationPreferences` interface (or import it from contracts if it exists there) and use `.$type<NotificationPreferences>()`.
- **Evidence**:

```typescript
notificationPreferences: jsonb("notification_preferences")
  .$type<{
    urgent_mentions: { email: boolean; push: boolean; inApp: boolean };
    todo_suggestions: { email: boolean; push: boolean; inApp: boolean };
    ...
  }>()
```

### Finding 4: AI model columns in `settings.table.ts` typed with hardcoded string union literals that duplicate `AI_CONFIG.MODELS`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/settings.table.ts:21-31`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: The `aiModelChatAssistant`, `aiModelReplyPrediction`, and `aiModelBackground` columns use inline `.$type<"claude-sonnet-4-6" | "claude-opus-4-6" | "claude-haiku-4-5">()` annotations. The same model IDs appear in `AI_CONFIG.MODELS` (which itself comes from `AI_MODEL_IDS` in `@slopweaver/contracts`). These three string literals are defined three times — in contracts, in `ai.constants.ts`, and now inline in the settings table. If a model ID changes, all three must be updated.
- **Suggestion**: Import `AIModel` from `ai.constants.ts` (or `AI_MODEL_IDS` from contracts) and use `.$type<AIModel>()` on the column definitions. Add a `CHECK` constraint in the table definition (as is done for `theme`) using the same union to keep DB and TS in sync.
- **Evidence**:

```typescript
aiModelChatAssistant: text("ai_model_chat_assistant").$type<
  "claude-sonnet-4-6" | "claude-opus-4-6" | "claude-haiku-4-5"
>();
```

### Finding 5: `featureId` in `feature-flags.table.ts` is plain `text` with no union type constraint

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/feature-flags.table.ts:17`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The `featureId` column is a plain `text` column with no type narrowing. Feature flags are presumably a closed set of known values (beta features, experiment flags, etc.). Allowing any string means inserts and lookups are unchecked.
- **Suggestion**: If there is a canonical list of feature flag IDs (likely exists as a constant somewhere in the codebase), import and apply it via `.$type<KnownFeatureId>()`. If the set is truly open-ended, add a JSDoc comment explaining this decision.
- **Evidence**:

```typescript
featureId: text("feature_id").notNull(),
```
