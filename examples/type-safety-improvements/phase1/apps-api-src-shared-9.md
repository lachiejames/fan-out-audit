# Audit: apps-api-src-shared-9

**Files inspected**: 8
**Findings**: 4

## Summary

Slice 9 covers settings, saved-filters, session-state, suggestion-accuracy-daily, and sync-related tables. The `sync-runs.table.ts` uses a plain `text` column for `status` with a comment listing known values — a clear candidate for a Drizzle enum or `.$type<>`. The `suggestion-accuracy-daily.table.ts` uses plain `text` for `dimensionType` with a comment listing valid values. `proactive-settings.table.ts` uses an `enabledChannels` array of untyped `text` strings.

## Findings

### Finding 1: `status` in `sync-runs.table.ts` is plain `text` with comment listing valid values

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/sync-runs.table.ts:47`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: The `status` column comment lists valid values: `"in_progress"`, `"paused"`, `"completed"`, `"failed"`, `"cancelled"`. The column is plain `text` with no `.$type<>` annotation or DB enum. Status-based queries and switch statements in the sync service read this value without any compile-time narrowing.
- **Suggestion**: Either add `.$type<"in_progress" | "paused" | "completed" | "failed" | "cancelled">()` for immediate TS benefit, or create a `syncRunStatusEnum = pgEnum(...)` in `enums.ts` and use it (preferred for DB-level enforcement). A `SyncRunStatus` type alias already exists as a pattern elsewhere (e.g., `WebhookEventStatus`, `SyncHealthStatus`).
- **Evidence**:

```typescript
status: text("status").notNull().default("in_progress"), // "in_progress", "paused", "completed", "failed", "cancelled"
```

### Finding 2: `dimensionType` in `suggestion-accuracy-daily.table.ts` is plain `text` with comment listing valid values

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/suggestion-accuracy-daily.table.ts:19`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The `dimensionType` column comment says `"overall" | "action_type" | "intent_category"` but the column is plain `text`. The analytics aggregation queries that group by `dimensionType` would benefit from compile-time narrowing.
- **Suggestion**: Add `.$type<"overall" | "action_type" | "intent_category">()` or create a `suggestionAccuracyDimensionTypeEnum`.
- **Evidence**:

```typescript
dimensionType: text("dimension_type").notNull(), // "overall" | "action_type" | "intent_category"
```

### Finding 3: `enabledChannels` in `proactive-settings.table.ts` is a `text` array with no element type constraint

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/proactive-settings.table.ts:35`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The `enabledChannels` column is `text("...").array()` with a default of `["desktop", "in_app"]`. The valid channel values correspond to `ProactiveNotificationChannel` values from `enums.ts` (`"desktop"`, `"mobile"`, `"in_app"`). There is no TS-level constraint on what strings can be inserted.
- **Suggestion**: Apply `.$type<ProactiveNotificationChannel[]>()` (importing `ProactiveNotificationChannel` from `enums.ts`) to enforce valid channel values at the TS layer.
- **Evidence**:

```typescript
enabledChannels: text("enabled_channels").array().default(["desktop", "in_app"]).notNull(),
```

### Finding 4: `dismissedSuggestionTypes` in `ProactiveSettingsMetadata` typed as `string[]` instead of a suggestion type union

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/proactive-settings.table.ts:26`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The `ProactiveSettingsMetadata` interface has `dismissedSuggestionTypes?: string[]`. The valid suggestion types correspond to `suggestionTypeEnum` values in `enums.ts` (`"urgent_mention"`, `"hidden_todo"`, etc.). Using `string[]` loses type safety for this field.
- **Suggestion**: Import `SuggestionType` (or `(typeof suggestionTypeEnum.enumValues)[number]`) from `enums.ts` and type this field as `SuggestionType[]` (or similar).
- **Evidence**:

```typescript
/** Dismissed suggestion types (user clicked "don't show this again") */
dismissedSuggestionTypes?: string[];
```
