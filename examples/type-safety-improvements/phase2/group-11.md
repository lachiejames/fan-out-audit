# Cross-Cutting Patterns (Group 11)

**Slices analyzed**: apps-api-src-shared-1, apps-api-src-shared-2, apps-api-src-shared-10, apps-api-src-shared-11, apps-api-src-shared-12, apps-api-src-shared-13, apps-api-src-shared-14, apps-api-src-shared-15, apps-api-src-shared-16, apps-api-src-shared-17, apps-api-src-shared-18, apps-api-src-shared-19

## Pattern 1: JSONB columns typed as `Record<string, unknown>` across Drizzle table definitions

- **Seen in**: shared-1, shared-2, shared-10, shared-11, shared-12, shared-13
- **Category**: missing-strict-typing
- **Combined impact**: high -- this is the single most pervasive type-safety gap in the shared layer, affecting at least 15 distinct JSONB columns across 10+ table files
- **What's happening**: Nearly every Drizzle table file that has a `jsonb` column types it as `.$type<Record<string, unknown>>()` or omits `$type<>` entirely. Affected columns include `cursorState` (sync-checkpoints), `metadata` (insights, token-usage, user-events), `replyMetadata`/`sourceMetadata` (todo), `aiSources`/`targetIdentifiers` (work-items), `details` (work-item-events), `externalResult` (work-item-runs), `payload` (lemon-squeezy-events, domain-events queue), `jobData` (failed-jobs), `sourceConfigOverride` (knowledge-source-import queue), and `WorkflowNode.data` (workflows). In most cases the actual runtime shape is known (per-platform cursors, per-event-type payloads, platform identifiers from contracts) but the type system does not express it.
- **Suggestion**: Audit each `Record<string, unknown>` JSONB column and replace it with the concrete type or discriminated union that already exists (or should exist) in the codebase. Prioritize columns where a discriminant column (e.g. `platform`, `eventType`, `activityType`) sits on the same table, since a discriminated union keyed on that column is straightforward. For columns that genuinely store open-ended data, add at minimum a partial interface with known fields plus an index signature.
- **Estimated scope**: 15+ JSONB columns across 10+ table files in `apps/api/src/shared/db/tables/`

## Pattern 2: Repeated `as Record<string, unknown>` cast pattern for duck-typed error access

- **Seen in**: shared-1, shared-14, shared-15, shared-16, shared-17, shared-18
- **Category**: type-cast
- **Combined impact**: medium -- at least 8 files independently repeat the same guard-then-cast boilerplate
- **What's happening**: Error extractor utilities (`ai-sdk-error.utils.ts`, `github-api-error.utils.ts`, `google-api-error.utils.ts`, `http-fetch-error.utils.ts`, `linear-error.utils.ts`, `postgres-error.utils.ts`, `microsoft-graph-error.utils.ts`, `notion-api-error.utils.ts`) and the AI usage utilities all follow the same pattern: check `typeof value === "object" && value !== null`, then cast to `Record<string, unknown>` with `as`. The guard is sufficient for runtime safety but does not produce a TypeScript type narrowing, so the `as` cast is required. Each file independently duplicates this boilerplate.
- **Suggestion**: Extract a shared `isRecord(value: unknown): value is Record<string, unknown>` type guard into `shared/utils/type-guards.utils.ts`. The `isObjectLike` guard in `postgres-error.utils.ts` already does exactly this. Promote it to a shared location, then replace all `as Record<string, unknown>` casts in the error extractors with the type guard call. This eliminates the casts entirely and centralizes the narrowing logic.
- **Estimated scope**: 8+ files, ~15-20 individual cast sites

## Pattern 3: Duplicate type definitions across sibling files

- **Seen in**: shared-12, shared-14, shared-15, shared-18
- **Category**: duplicate-type
- **Combined impact**: medium -- at least 4 independent instances of the same type defined in two places
- **What's happening**: Multiple pairs of files define the same type independently: (1) `ErrorStatusCode` union in both `error-mapper.factory.ts` and `error-response.utils.ts`, (2) `StandardErrorBody` interface in the same two files, (3) `VocabularySource` union duplicated inline in `user-vocabulary.table.ts` and `user-vocabulary-aliases.table.ts`, (4) `DatabaseError` interface in both `message.errors.ts` and `database.errors.ts` (name collision), (5) `OAuthPlatformType` locally redefined with a manual sync comment instead of importing from contracts. Each duplication is an independent divergence risk.
- **Suggestion**: For each pair: define the type once in the canonical location and import it elsewhere. Specifically: (a) keep `ErrorStatusCode` and `StandardErrorBody` in `error-response.utils.ts` only, import into factory; (b) extract `VocabularySource` to a shared types file; (c) rename `message.errors.ts`'s `DatabaseError` to `MessageDatabaseError`; (d) replace `OAuthPlatformType` with `PlatformId` from contracts (or a derived subset).
- **Estimated scope**: 5 duplicate type pairs across 8 files

## Pattern 4: `Record<string, ...>` with string keys where a closed union exists

- **Seen in**: shared-1, shared-2, shared-16
- **Category**: record-weakening
- **Combined impact**: medium -- affects configuration maps and constant lookups where typo-safety matters
- **What's happening**: Several configuration objects and constant maps use `Record<string, T>` as their key type even though the valid keys form a known closed set: (1) `endpointClasses` uses `Record<string, EndpointClassConfig>` but only has `"read"` and `"write"` keys, (2) `HIGH_FREQUENCY_JOB_TYPES` is `ReadonlySet<string>` instead of `ReadonlySet<ScheduledJobType>`, (3) `MIME_TYPE_EXTRACTORS` is `Record<string, ExtractorFn>` with a fixed set of MIME keys, (4) `OUTBOUND_RATE_LIMITS` requires an `as` cast because `Object.fromEntries` returns `Record<string, T>`.
- **Suggestion**: Replace `string` keys with the appropriate literal union in each case. For `Object.fromEntries` results, follow with `satisfies Record<UnionKey, T>` instead of `as`. For sets and maps, use the union type directly as the generic parameter.
- **Estimated scope**: 4+ instances across config, constants, and utility files

## Pattern 5: Plain `text` columns with known literal values documented only in comments

- **Seen in**: shared-11, shared-13
- **Category**: missing-strict-typing
- **Combined impact**: low-medium -- allows invalid values to be written to the database without compile-time error
- **What's happening**: Several Drizzle table columns use `text()` for fields that have a known, closed set of valid values, with the valid values documented only in inline comments: (1) `aiSuggestion` in `triage-items.table.ts` accepts `"archive" | "reply" | "todo" | "review"` per comment but is typed as `string | null`, (2) `status` and `registrationType` in `webhook-registrations.table.ts` similarly document valid values in comments but accept any string. These are distinct from the JSONB pattern because the fix is simpler: either a `pgEnum` or a `.$type<>` annotation.
- **Suggestion**: For each affected column, either define a `pgEnum` (if the values are stable and warrant a database-level constraint) or at minimum add `.$type<"value1" | "value2">()` to get compile-time enforcement. The `pgEnum` approach is preferred when the same values appear in multiple tables (e.g. `aiSuggestion` appears in both `triage-items` and `triage-overrides`).
- **Estimated scope**: 4+ columns across 3 table files

## Pattern 6: `any` usage in generic type infrastructure

- **Seen in**: shared-14, shared-15
- **Category**: any-usage
- **Combined impact**: low -- confined to type-level utility code, not runtime values
- **What's happening**: The `error-mapper.factory.ts` uses `any` in several generic type parameter positions (`(...args: any[]) => infer R`, `Record<string, any>`, `TError extends Record<string, any>`). The `ai.types.ts` file has `"low" | "medium" | "high" | string` which collapses to `string` (functionally similar to `any` for that field). These are in type-level inference positions rather than runtime code, but they still suppress type checking on inputs.
- **Suggestion**: Replace `Record<string, any>` with `Record<string, unknown>` where the value type is not read in the implementation body. Replace `any[]` with `unknown[]` in function signature constraints. Remove `| string` from the severity union to preserve literal type narrowing.
- **Estimated scope**: 2 files, ~5 instances
