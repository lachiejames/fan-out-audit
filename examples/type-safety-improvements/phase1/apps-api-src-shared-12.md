# Audit: apps-api-src-shared-12

**Files inspected**: 8
**Findings**: 3

## Summary

Most tables in this slice are well-typed. The primary issues are: (1) a `CalendarProvider` type defined locally in `user-calendar-config.table.ts` that should be shared or imported from contracts, (2) two `jsonb` columns typed as `Record<string, unknown>` where more specific types exist or could be inferred, and (3) a `source` column in both `user-vocabulary.table.ts` and `user-vocabulary-aliases.table.ts` that uses an identical inline `$type<>` union — a clear duplicate type definition.

## Findings

### Finding 1: Duplicate `source` union type across `user-vocabulary` and `user-vocabulary-aliases`

- **File**: `apps/api/src/shared/db/tables/user-vocabulary.table.ts:21-33` and `apps/api/src/shared/db/tables/user-vocabulary-aliases.table.ts:22-34`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: Both tables define the exact same 8-member string union inline as a `$type<>` for the `source` column:
  ```
  "manual" | "seed_profile" | "seed_entity" | "seed_integration" | "seed_content" | "seed_knowledge" | "draft_correction" | "transcript_correction"
  ```
  If this union ever needs to change, it must be updated in two places, which risks divergence.
- **Suggestion**: Extract the union to a named exported type (e.g. `VocabularySource`) in `enums.ts` or a shared types file, and import it in both tables.
- **Evidence**:

  ```ts
  // user-vocabulary.table.ts
  source: text("source").$type<"manual" | "seed_profile" | "seed_entity" | "seed_integration" | "seed_content" | "seed_knowledge" | "draft_correction" | "transcript_correction">().notNull(),

  // user-vocabulary-aliases.table.ts (identical)
  source: text("source").$type<"manual" | "seed_profile" | "seed_entity" | "seed_integration" | "seed_content" | "seed_knowledge" | "draft_correction" | "transcript_correction">().notNull(),
  ```

### Finding 2: `CalendarProvider` local type that may duplicate a contracts type

- **File**: `apps/api/src/shared/db/tables/user-calendar-config.table.ts:16`
- **Category**: sdk-type-duplication
- **Impact**: low
- **Description**: `CalendarProvider = "google" | "microsoft" | "apple"` is defined locally in the table file. If `@slopweaver/contracts` already defines a calendar provider union or if the integrations layer has a `PlatformId` that covers these values, this is a duplicate. Even if contracts don't have it yet, placing this type in the table file rather than a shared types file makes it invisible to the application layer.
- **Suggestion**: Move `CalendarProvider` to `shared/types/` or export it from `@slopweaver/contracts` alongside other platform identifiers. Import it into the table rather than defining it inline.
- **Evidence**:
  ```ts
  export type CalendarProvider = "google" | "microsoft" | "apple";
  ```

### Finding 3: `metadata` jsonb column in `user-events.table.ts` typed as `Record<string, unknown>`

- **File**: `apps/api/src/shared/db/tables/user-events.table.ts:21`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The `metadata` column on `user_events` is typed as `Record<string, unknown>`. User events are typed by `activityTypeEnum`, so each event type likely has a predictable metadata shape. A discriminated union keyed on `activityTypeEnum` values would allow safe narrowing at read-sites.
- **Suggestion**: Define per-activity-type metadata interfaces (or at minimum a partial record like `{ contentId?: string; platform?: string; [key: string]: unknown }`) and replace `Record<string, unknown>` with the named type.
- **Evidence**:
  ```ts
  metadata: jsonb("metadata").$type<Record<string, unknown>>(),
  ```

## No additional findings

`triage-sessions.table.ts`, `user-actions.table.ts`, `user-notification-preferences.table.ts`, `user-push-tokens.table.ts` are clean: they use proper enum columns, no `any`, no unsafe casts.
