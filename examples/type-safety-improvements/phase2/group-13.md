# Cross-Cutting Patterns (Group 13)

**Slices analyzed**: apps-app-src-components-2, apps-app-src-components-14, apps-app-src-components-15, apps-app-src-components-16, apps-app-src-components-17, apps-app-src-components-18, apps-app-src-components-19, apps-app-src-components-20, apps-app-src-components-21, apps-app-src-components-22, apps-app-src-components-23, apps-app-src-components-24

## Pattern 1: `Record<string, T>` lookup maps where contract union keys exist

- **Seen in**: slice-2 (iconMap, updateSettings), slice-14 (CORE_INTEGRATION_COPY, platformSyncStates), slice-16 (fileStatusConfig, TERMINAL_STATUS_BADGES), slice-18 (DETAIL_FIELD_LABELS, WORK_ITEM_TYPE_LABELS, TERMINAL_STATUS_BADGES), slice-21 (PROVIDER_LABELS, ITEM_TYPE_ICONS, CATEGORY_COLORS, STATUS_BADGES), slice-22 (PHASE_LABELS), slice-23 (formatPlatformName)
- **Category**: record-weakening
- **Combined impact**: high -- this is the single most pervasive type-safety gap in the frontend components
- **What's happening**: Constant lookup maps (labels, icons, badges, colors, config) are typed as `Record<string, T>` even though the valid key set is a known union from `@slopweaver/contracts` (e.g., `PlatformId`, `WorkItemType`, `WorkItemStatus`, `SyncPhase`, `KnowledgeItemType`, `ImportedKnowledgeCategory`). This means misspelled keys compile silently, new union members added to contracts do not trigger compile errors for missing map entries, and lookups return `T` instead of `T | undefined`, hiding potential runtime misses.
- **Suggestion**: Systematically replace `Record<string, T>` with `Partial<Record<ContractUnion, T>>` (when the map covers a subset) or `Record<ContractUnion, T>` (when exhaustiveness is desired). A single lint rule or codemod scanning for `Record<string,` in component files could identify all remaining instances.
- **Estimated scope**: 15+ constant maps across at least 12 files in `apps/app/src/components/`

## Pattern 2: Unsafe `as SomeType` casts on values from `unknown` or `string` sources

- **Seen in**: slice-2 (ref casts), slice-14 (as PlatformId, as readonly string[]), slice-15 (as Components, as Error & statusCode, as ErrorResponse), slice-16 (double cast platform as "google-docs", meta["attendees"] as string[]), slice-17 (property as Record<string, unknown>), slice-19 (workItem.type as ..., workItem.status as keyof typeof ...), slice-21 (sourceConfig as inline struct, captureMetadata as inline struct), slice-24 (as AIModelId, as ThinkingMode, as TaskStatus[], as TaskPriority[], as TaskEffort[])
- **Category**: type-cast
- **Combined impact**: high -- scattered across nearly every slice, representing the largest count of individual findings
- **What's happening**: Components cast values from `unknown`, `string`, or broad union types to narrow types without runtime validation. Common patterns: (a) DOM `event.target.value as ContractType` on select handlers, (b) `workItem.field as NarrowType` where the contract type is broader than needed, (c) inline structural casts (`as { field?: string }`) on `unknown` config/metadata fields, (d) array literal casts (`["a", "b"] as MyType[]`). These bypass the type system and will not catch runtime mismatches.
- **Suggestion**: Three systematic fixes: (1) For DOM select values, create `isXType()` type guards per contract union and validate on change. (2) For contract fields that are too broad (e.g., `workItem.type: string` instead of `WorkItemType`), fix the type at the contract/response level so downstream casts become unnecessary. (3) For literal array casts, use `as const satisfies T[]` to get both literal inference and compile-time validation. (4) For `unknown` config/metadata fields, define typed discriminated shapes in contracts rather than casting inline.
- **Estimated scope**: 20+ individual cast sites across 15+ files

## Pattern 3: Duplicate type definitions and derivations across sibling files

- **Seen in**: slice-2 (ReasoningState in two files), slice-15 (BackendIntegrationRecord / BackendIntegration), slice-19 (inline "draft" | "sending" | "sent" | "error" vs QueueSendState), slice-21 (PROVIDER_LABELS derived identically in two knowledge-source files)
- **Category**: duplicate-type
- **Combined impact**: medium -- each instance is a maintenance risk where changes must be made in two places
- **What's happening**: The same type or derived constant is defined independently in multiple files. This includes exact type duplicates (ReasoningState), near-identical interfaces (BackendIntegrationRecord vs BackendIntegration), inline union literals that duplicate an existing exported type (QueueSendState), and identical derivation logic (PROVIDER_LABELS from KNOWLEDGE_SOURCE_PROVIDER_META).
- **Suggestion**: For each duplicate pair: extract to a single shared location and import. For inline literals duplicating an existing type, replace with the named type import. A search for identical type shapes across the components directory would likely surface additional instances beyond those found in this audit.
- **Estimated scope**: 4 confirmed duplicate pairs across 8 files; likely more undiscovered

## Pattern 4: `string`-typed platform identifiers instead of `PlatformId`

- **Seen in**: slice-14 (CORE_INTEGRATION_COPY keys, platformSyncStates keys, platformId as PlatformId cast), slice-19 (workItem.type as narrow union), slice-23 (formatPlatformName accepts string)
- **Category**: record-weakening (overlaps with type-cast)
- **Combined impact**: medium -- `PlatformId` is the canonical type but many component props and helpers accept raw `string`
- **What's happening**: Multiple components accept `string` where `PlatformId` from `@slopweaver/contracts` is the correct type. This forces downstream code to cast (`as PlatformId`) or to use `isPlatformId()` guards that would be unnecessary if the prop boundary were correctly typed. The pattern suggests that either the data source (API response types, route params) provides `string` instead of `PlatformId`, or component authors defaulted to `string` for convenience.
- **Suggestion**: Trace the `string` back to its source. If API response types use `string` for platform fields, fix them at the contract level. If route params are the source, add validation at the route boundary and propagate `PlatformId` from there. This eliminates casts and guards throughout the component tree.
- **Estimated scope**: 5+ component props/helpers, plus associated casts; root cause may be 1-2 contract types

## Pattern 5: Missing type guards for platform/type narrowing functions

- **Seen in**: slice-14 (isPlatformId guard needed for PLATFORMS access), slice-16 (isResourceCardPlatform returns boolean instead of type predicate), slice-19 (MINIMAL_ACTION_TYPES.has() does not narrow)
- **Category**: type-cast (root cause)
- **Combined impact**: medium -- missing type predicates are the root cause of multiple downstream casts
- **What's happening**: Guard functions like `isResourceCardPlatform()` and `isPlatformId()` either return `boolean` instead of a type predicate (`platform is PlatformId`), or are not used where they should be. This forces every call site to add a manual cast after the guard check, which is both noisy and unsafe.
- **Suggestion**: Audit all `is*` functions in `@slopweaver/contracts` and ensure they return proper type predicates. Similarly, for `Set.has()` checks on known unions, wrap them in a type predicate helper. Each fix eliminates casts at every call site that uses the guard.
- **Estimated scope**: 3-5 guard functions in contracts; fixes cascade to 10+ cast sites in components

## Pattern 6: Unsafe error handling casts in catch blocks

- **Seen in**: slice-15 (error as Error & { statusCode?: number }, error.body as ErrorResponse)
- **Category**: type-cast
- **Combined impact**: low -- only one slice but the pattern likely exists elsewhere
- **What's happening**: Catch blocks cast `unknown` errors to structured types without runtime validation. The `error as Error & { statusCode?: number }` pattern assumes the caught value is an Error instance, which is not guaranteed in JavaScript.
- **Suggestion**: Use `instanceof` checks and `"property" in error` guards before accessing error fields. The codebase already has `extractErrorMessage` and `ApiResponseError` -- ensure all catch blocks use these utilities consistently.
- **Estimated scope**: 2 instances confirmed in this group; likely more in other component slices
