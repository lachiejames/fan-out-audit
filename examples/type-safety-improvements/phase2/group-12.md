# Cross-Cutting Patterns (Group 12)

**Slices analyzed**: apps-api-src-shared-3, apps-api-src-shared-4, apps-api-src-shared-5, apps-api-src-shared-6, apps-api-src-shared-7, apps-api-src-shared-8, apps-api-src-shared-9, apps-app-src-components-1, apps-app-src-components-10, apps-app-src-components-11, apps-app-src-components-12, apps-app-src-components-13

## Pattern 1: Untyped jsonb columns across Drizzle table definitions

- **Seen in**: apps-api-src-shared-3, apps-api-src-shared-4, apps-api-src-shared-5, apps-api-src-shared-6, apps-api-src-shared-7, apps-api-src-shared-8
- **Category**: missing-strict-typing
- **Combined impact**: high -- this is the single largest type-safety gap in the backend; every untyped jsonb column forces downstream code to cast or assert
- **What's happening**: Across six shared slices, at least 25 jsonb columns lack `.$type<>()` annotations entirely (returning `unknown` from Drizzle), and another 5-6 use `.$type<Record<string, unknown>>()` which is only marginally better. Key examples: `knowledge-source-revisions.table.ts` alone has 8 untyped jsonb columns; `predictions.table.ts` has 2 critical AI output columns untyped; `feedback.table.ts`, `suggestion-feedback.table.ts`, and `suggestion-presentations.table.ts` all have untyped suggestion/action payloads. Meanwhile, `chat-messages.table.ts` demonstrates the gold-standard pattern with `.$type<UiPart[]>()` imported from contracts.
- **Suggestion**: Systematically add `.$type<T>()` to every jsonb column in the DB layer. Prioritize columns that are read in application logic (predictions, suggestions, knowledge source progress). For each column, either import an existing type from `@slopweaver/contracts` or define a new interface in the table file. The gold-standard pattern from `chat-messages.table.ts` should be the template.
- **Estimated scope**: ~30 jsonb columns across ~15 table files in `apps/api/src/shared/db/tables/`

## Pattern 2: `Record<string, unknown>` used for metadata columns instead of typed interfaces

- **Seen in**: apps-api-src-shared-3, apps-api-src-shared-4, apps-api-src-shared-5, apps-api-src-shared-7, apps-api-src-shared-8
- **Category**: record-weakening
- **Combined impact**: high -- metadata columns are among the most frequently accessed in application code, and every access requires unsafe property lookups or casts
- **What's happening**: At least 6 `metadata` columns across different tables use `.$type<Record<string, unknown>>()`: `action-transactions`, `contents`, `knowledge-items`, `proactive-notification`, plus the untyped `metadata` columns in `feedback` and `knowledge-source-import-events`. In each case, comments or downstream code reveal that the metadata has a known, specific shape (action type + model used, sender details, extraction timestamps, trigger context, etc.), but the type declaration is the maximally permissive `Record<string, unknown>`.
- **Suggestion**: For each metadata column, define a named interface (e.g., `ActionTransactionMetadata`, `ContentMetadata`, `KnowledgeItemMetadata`) with the known optional fields. Even partial types that document the primary keys are a significant improvement over `Record<string, unknown>`. These interfaces can live alongside their table definitions or in a shared `db/types/` directory.
- **Estimated scope**: ~6-8 metadata columns across 6+ table files

## Pattern 3: Plain `text` columns with comments listing valid values instead of `.$type<>()` or pgEnum

- **Seen in**: apps-api-src-shared-4, apps-api-src-shared-5, apps-api-src-shared-6, apps-api-src-shared-8, apps-api-src-shared-9
- **Category**: missing-strict-typing
- **Combined impact**: medium -- these columns accept any string at both the DB and TS level, despite having a closed set of valid values documented in comments
- **What's happening**: Multiple `text` columns have comments listing their valid values but no type narrowing: `purpose` in `anthropic-files` ("core-profile", "knowledge", etc.), `status` in `invoices` ("paid", "pending", "refunded"), `status` in `sync-runs` ("in_progress", "paused", etc.), `dimensionType` in `suggestion-accuracy-daily` ("overall", "action_type", "intent_category"), `featureId` in `feature-flags`, and `status` in `pending-integrations` (which at least has a local type alias). The codebase already uses `pgEnum` extensively in `enums.ts` for similar cases, showing the preferred pattern exists but is not consistently applied.
- **Suggestion**: For each column, either (a) add a `pgEnum` in `enums.ts` for DB-level enforcement (preferred), or (b) at minimum add `.$type<"value1" | "value2" | ...>()` for TS-level narrowing. Batch these changes since they follow the same mechanical pattern.
- **Estimated scope**: ~6-8 columns across 6 table files

## Pattern 4: `Record<string, React.ReactNode>` icon maps with closed key sets

- **Seen in**: apps-app-src-components-11, apps-app-src-components-12
- **Category**: record-weakening
- **Combined impact**: medium -- silently returns `undefined` for unknown keys, which can cause rendering bugs
- **What's happening**: At least 4 icon map objects across inbox and suggestion components use `Record<string, React.ReactNode>` where the actual keys are a small, fixed set (e.g., "email", "calendar", "slack"). The maps in `compound-suggestion-chip.tsx`, `suggestion-chips.tsx`, `suggestion-step-card.tsx`, and potentially others all follow this pattern. An unknown key produces `undefined` with no compile-time error.
- **Suggestion**: Define each icon map using `as const` satisfies and derive the key type: `const ICON_MAP = { email: <Icon/>, ... } as const satisfies Record<string, React.ReactNode>`. Then type-narrow the lookup input to `keyof typeof ICON_MAP`. If multiple components share icons, extract a shared icon registry type.
- **Estimated scope**: ~4-5 icon map instances across 4 files in `apps/app/src/components/`

## Pattern 5: `string` used where contract-defined union types exist (PlatformId, AIModel, enums)

- **Seen in**: apps-api-src-shared-3, apps-api-src-shared-5, apps-api-src-shared-9, apps-app-src-components-10, apps-app-src-components-11
- **Category**: missing-strict-typing
- **Combined impact**: high -- `string` in place of known unions defeats exhaustiveness checking for platform-conditional logic, AI model selection, and enum-backed fields
- **What's happening**: Three distinct sub-patterns converge here: (1) `platform` fields typed as `string` instead of `PlatformId` in multiple inbox components (message-card.tsx and others), (2) AI model columns in `settings.table.ts` using inline string literal unions that duplicate `AI_MODEL_IDS` from contracts, (3) fields like `enabledChannels`, `dismissedSuggestionTypes`, `urgency`, and `channel` using `string` or `string[]` instead of importing the corresponding enum types from `enums.ts`. In each case, the canonical type already exists in either `@slopweaver/contracts` or `enums.ts` but is not used.
- **Suggestion**: Audit all `string` fields that correspond to known enums or contract types. Replace with imports from the canonical source. This is a mechanical refactor: find the `string` usage, identify the canonical type, and swap the import. The AI model duplication (inline literals in settings.table.ts vs. `AI_CONFIG.MODELS` vs. contracts) should be collapsed to a single source in contracts.
- **Estimated scope**: ~15-20 fields across 10+ files spanning both API tables and frontend components

## Pattern 6: Duplicated inline type shapes instead of shared named types

- **Seen in**: apps-api-src-shared-5, apps-app-src-components-1, apps-app-src-components-11
- **Category**: duplicate-type
- **Combined impact**: medium -- identical shapes defined in multiple places drift independently and force maintenance of multiple definitions
- **What's happening**: Three concrete instances: (1) In `settings.table.ts`, the `{ accuracy: number; accepted: number; shown: number }` calibration bucket shape is repeated three times inline for `byActionType`, `byIntentCategory`, and `bySenderDomain`. (2) `FileValidationResult` in the app may overlap with `ValidationResult` from `@slopweaver/ui`. (3) Inbox filter/sort values are implicitly duplicated between `inbox-filter-bar.tsx` and `mobile-filter-sheet.tsx` with no shared type source. Each duplication increases the risk of one copy diverging from another.
- **Suggestion**: Extract repeated shapes into named types: `CalibrationBucket` for the settings table, a shared `InboxFilterConfig` for inbox components. For `FileValidationResult`, verify structural compatibility with the UI package's `ValidationResult` and consolidate if possible.
- **Estimated scope**: ~5-6 instances across 4-5 files

## Pattern 7: Unsafe `as` casts on string-split results

- **Seen in**: apps-app-src-components-11 (two files), apps-app-src-components-13
- **Category**: type-cast
- **Combined impact**: medium -- runtime failures are silent if the input string format changes; the cast also duplicated across files
- **What's happening**: `v.split("-") as [SortField, SortDirection]` appears in both `inbox-filter-bar.tsx` and `mobile-filter-sheet.tsx`. Separately, `pre-connect-platform-content.ts` has ~10 instances of `as Record<string, string>` on object literals that are entirely redundant. The string-split casts are the more dangerous variant since they assert a tuple shape on an unchecked runtime value.
- **Suggestion**: For the string-split pattern: extract a `parseSortValue()` helper with runtime validation into a shared `inbox-filter.utils.ts` file. For the `as Record<string, string>` casts: remove them entirely since the function return type already provides the constraint.
- **Estimated scope**: 2 files for string-split casts (~2 instances), 1 file for redundant Record casts (~10 instances)

## Pattern 8: Platform-specific type aliases that are all structurally identical

- **Seen in**: apps-api-src-shared-3, apps-api-src-shared-4
- **Category**: record-weakening
- **Combined impact**: medium -- 15 platform cursor aliases all equal `Record<string, unknown>`, giving a false sense of type safety; similar pattern with structurally identical `Record<string, number>` aliases in behavioral-fingerprint
- **What's happening**: `sync-checkpoints.table.ts` defines `CursorState = Record<string, unknown>` and then 15 platform aliases (`GmailCursorState`, `SlackCursorState`, etc.) that are all identical. `behavioral-fingerprint.table.ts` defines `FormalityByRecipient` and `ResponseSpeedByRecipient` as identical `Record<string, number>`. These aliases provide nominal differentiation but zero structural type safety -- they are fully interchangeable.
- **Suggestion**: For cursor states: either define concrete per-platform cursor shapes (e.g., `GmailCursorState = { historyId: string; nextPageToken?: string }`) or remove the aliases entirely and use `CursorState` with runtime Zod validation at platform boundaries. For behavioral fingerprint: consider branded types if the distinction matters, otherwise collapse to a single alias.
- **Estimated scope**: 15 cursor aliases in 1 file, 2 fingerprint aliases in 1 file
