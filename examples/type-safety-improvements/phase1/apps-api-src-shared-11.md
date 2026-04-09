# Audit: apps-api-src-shared-11

**Files inspected**: 8
**Findings**: 3

## Summary

The table files in this slice are generally well-typed, using Drizzle's `$inferSelect`/`$inferInsert` pattern throughout and enum columns instead of raw strings where appropriate. The main findings are two `jsonb` columns typed as the generic `Record<string, unknown>` (in `todo.table.ts`) where a more specific application type would be safer, and one plain-text field that should be an enum column (`aiSuggestion` in `triage-items.table.ts`).

## Findings

### Finding 1: `replyMetadata` and `sourceMetadata` jsonb columns typed as `Record<string, unknown>`

- **File**: `apps/api/src/shared/db/tables/todo.table.ts:30-33`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: Both `replyMetadata` and `sourceMetadata` are typed as `Record<string, unknown>` via `.$type<Record<string, unknown>>()`. This is the weakest useful type for JSONB — any consumer must narrow the value before use. `sourceMetadata` in particular likely carries a known shape (origin platform, external IDs, etc.) that could be codified at the schema level, providing compile-time safety for both writers and readers.
- **Suggestion**: Define a discriminated union or concrete interface (e.g. `TodoSourceMetadata`) for `sourceMetadata`, and if `replyMetadata` has a discoverable shape (e.g. `{ threadId: string; platform: string }`) express it explicitly. Use `.$type<TodoSourceMetadata>()`.
- **Evidence**:
  ```ts
  replyMetadata: jsonb("reply_metadata").$type<Record<string, unknown>>(),
  sourceMetadata: jsonb("source_metadata").$type<Record<string, unknown>>().default({}).notNull(),
  ```

### Finding 2: `aiSuggestion` column is untyped `text` in `triage-items.table.ts`

- **File**: `apps/api/src/shared/db/tables/triage-items.table.ts:21`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `aiSuggestion` is stored as plain `text` with an inline comment documenting the valid values (`"archive" | "reply" | "todo" | "review"`). The same field in `triage-overrides.table.ts` is also `text`. This pattern relies on comment documentation rather than type enforcement. Without an enum or `$type<>`, invalid values can be written without a TypeScript error.
- **Suggestion**: Either define a `pgEnum` (e.g. `triageActionEnum`) and use it for both `triage-items` and `triage-overrides`, or at minimum apply `.$type<"archive" | "reply" | "todo" | "review">()` to the column so the inferred TypeScript type is a literal union rather than `string | null`.
- **Evidence**:
  ```ts
  aiSuggestion: text("ai_suggestion"), // "archive", "reply", "todo", "review"
  ```

### Finding 3: `metadata` jsonb column in `token-usage.table.ts` typed as `Record<string, unknown>`

- **File**: `apps/api/src/shared/db/tables/token-usage.table.ts:37`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The `metadata` column is documented as containing "batch size, telemetry data" but is typed as `Record<string, unknown>`. A concrete type (or at least a partial known shape) would make call-sites safer.
- **Suggestion**: Define an interface such as `TokenUsageMetadata { batchSize?: number; [key: string]: unknown }` and apply `.$type<TokenUsageMetadata>()`. This narrows known fields while still allowing extension.
- **Evidence**:
  ```ts
  metadata: jsonb("metadata").$type<Record<string, unknown>>(),
  ```

## No additional findings

`templates.table.ts`, `todo-proposal-actions.table.ts`, `todo-proposals.table.ts`, `training-examples.table.ts`, and `triage-overrides.table.ts` are clean: they use proper enum columns, have no `any`, no unsafe casts, and no duplicate type definitions within this slice.
