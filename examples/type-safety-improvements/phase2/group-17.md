# Cross-Cutting Patterns (Group 17)

**Slices analyzed**: apps-app-src-sentry.ts, apps-app-src-shared-app-1, apps-app-src-shared-app-2, apps-app-src-shared-app-3, apps-app-src-types, apps-app-src-utils-1, apps-app-src-utils-2, apps-app-src-utils-3, packages-contracts-src-analytics, packages-contracts-src-config, packages-contracts-src-contracts-1, packages-contracts-src-contracts-10

## Pattern 1: `Record<string, unknown>` for structured data with cascading `as` casts

- **Seen in**: apps-app-src-utils-1 (InboxQueryData, AI transparency labels), apps-app-src-utils-2 (metadata in inbox-data.utils, search-result.transformer, CATEGORY_COLORS), apps-app-src-utils-3 (queue audit details, todo sourceMetadata), apps-app-src-shared-app-1 (PaywallContext event detail), packages-contracts-src-contracts-1 (MODEL_TIER_LABELS, TASK_TYPE_LABELS), packages-contracts-src-contracts-10 (Jira customFields)
- **Category**: record-weakening
- **Combined impact**: high -- this is the single most pervasive type-safety issue across the codebase
- **What's happening**: Data with known field names and types is typed as `Record<string, unknown>` (or `Record<string, string>`). Every consumer must then cast fields individually via `details["platform"] as string`, `metadata["snippet"] as string`, etc. These casts silently return `undefined` when the field is missing or has the wrong runtime type, with no compiler warning. The pattern appears in at least 8 distinct utility files spanning inbox data, queue audits, todo metadata, search results, and analytics labels.
- **Suggestion**: Define typed interfaces for each known data shape (e.g., `TodoSourceMetadata`, `QueueAuditDetails` discriminated by action type, `InboxMessageMetadata`). For label/color lookup maps where keys come from a known union, use `Record<KnownKey, string>` or `Partial<Record<KnownKey, string>>` instead of `Record<string, string>`. This eliminates all downstream `as` casts and enables exhaustiveness checking when new keys are added.
- **Estimated scope**: 15-20 files, 40+ individual cast sites

## Pattern 2: `platform` fields typed as bare `string` instead of `PlatformId`

- **Seen in**: apps-app-src-shared-app-3 (WizardIntent.platform), apps-app-src-utils-3 (formatPlatformName switch), packages-contracts-src-analytics (IntegrationEventProperties.platform, SyncEventProperties.platform), packages-contracts-src-contracts-1 (agentToolMetadataSchema.platforms, toolActionSchema oauth_popup.platform)
- **Category**: missing-strict-typing
- **Combined impact**: high -- the `PlatformId` union and `isPlatformId` guard already exist in contracts but are not used consistently
- **What's happening**: The codebase has a canonical `PlatformId` union type and `PLATFORMS` registry in `@slopweaver/contracts`, yet many files declare platform fields as plain `string`. This allows typos like `"gmailz"` to pass both compile-time and Zod runtime checks. In the frontend, `formatPlatformName` reimplements a platform-name switch that duplicates the `PLATFORMS` registry display names. In contracts, both analytics event properties and agent tool schemas accept arbitrary strings for platforms.
- **Suggestion**: Systematically replace `platform: string` with `platform: PlatformId` (or `PlatformId | (string & {})` for forward-compat). In Zod schemas, use `z.enum([...PLATFORM_IDS])` or `z.string().refine(isPlatformId)`. Replace the `formatPlatformName` switch with a lookup into the existing `PLATFORMS` registry.
- **Estimated scope**: 8-10 files across contracts and apps/app

## Pattern 3: Local type definitions that duplicate or drift from `@slopweaver/contracts`

- **Seen in**: apps-app-src-shared-app-1 (local MessagePriority), apps-app-src-types (re-exported Citation/CitationPlatform), apps-app-src-utils-1 (local BehavioralFingerprint), apps-app-src-utils-2 (local ParsedGmailEmail, ToolCall import from wrong path), apps-app-src-utils-3 (inline transformTodo parameter type, local TodoQueryParams.status, local ToolCallStatus), packages-contracts-src-contracts-1 (AnalyticsResponse deprecated alias, WorkItemRevision duplicate)
- **Category**: duplicate-type
- **Combined impact**: high -- at least 10 distinct duplicate type definitions across the frontend and contracts
- **What's happening**: The frontend defines local interfaces/types that mirror shapes already exported from `@slopweaver/contracts` (or from other canonical sources within the app). Examples: `MessagePriority` redefined as a local union, `BehavioralFingerprint` defined locally with 20+ fields instead of deriving from the API type, `transformTodo` accepting a 20-field inline object instead of `TodoResponse`, `TodoQueryParams.status` hardcoding status literals instead of importing `TodoStatus`, and `ToolCallStatus` defined in two separate files. When contracts evolve (add a priority tier, rename a field, add a status), these local types silently diverge.
- **Suggestion**: For each duplicate, either (a) import the canonical type directly, (b) derive the local type using `Pick`/`Omit`/intersection from the contract type, or (c) if the local type intentionally differs (e.g., UI projection with scaled values), formally derive it with `Omit<ApiType, ...> & { ... }` and add a comment explaining the divergence. Remove deprecated aliases (`AnalyticsResponse`, `WorkItemRevisionResponse`) after migrating consumers.
- **Estimated scope**: 10-12 files, 10+ distinct type definitions to consolidate

## Pattern 4: `as SomeType` casts on string-to-union lookups instead of type predicates

- **Seen in**: apps-app-src-shared-app-1 (priority as MessagePriority, category as TaskCategory, platformId as PlatformId), apps-app-src-utils-3 (modelId as AIModelId), apps-app-src-utils-1 (tone as "formal" | "casual" | "balanced")
- **Category**: type-cast
- **Combined impact**: medium -- consistent anti-pattern across all config/lookup utilities
- **What's happening**: When looking up a value in a `Record<UnionKey, Value>` map, an incoming `string` parameter is cast to the union key type via `as`. The lookup then returns `Value` according to TypeScript, but at runtime returns `undefined` if the string is not a valid key. The return type does not reflect the `undefined` possibility. This pattern appears in priority config, task category config, platform config, AI model info, and tone lookups.
- **Suggestion**: Replace each `as UnionType` cast with a type predicate guard function (e.g., `isMessagePriority(value): value is MessagePriority`). Several guards already exist in the codebase (`isPlatformId`, `isTaskCategory`) but are not used at the lookup sites. The guard pattern narrows the type safely, and the else branch can return a defined fallback or `undefined` with an honest return type.
- **Estimated scope**: 6-8 files, 8-10 cast sites

## Pattern 5: `import.meta.env` values cast with `as` instead of validated

- **Seen in**: apps-app-src-sentry.ts (SLOPWEAVER_ENV as string), apps-app-src-shared-app-2 (VITE_APP_DISTRIBUTION as AppDistribution), packages-contracts-src-config (import.meta as { env?: Record<string, string> })
- **Category**: type-cast
- **Combined impact**: low -- build-time constants with limited blast radius, but a systemic pattern
- **What's happening**: Environment variables read from `import.meta.env` are cast to their expected types (`string`, `AppDistribution`, etc.) without runtime validation. If a build misconfigures an env var, the invalid value passes silently through the cast and may cause subtle downstream bugs.
- **Suggestion**: Create a shared `validateEnv` utility (or use Zod) that validates env vars once at startup and returns typed values. The `@slopweaver/env` package may already provide this -- if so, use it consistently. For the `import.meta` cast in non-Vite contexts, the defensive pattern is acceptable but should be documented with a comment.
- **Estimated scope**: 3-5 files

## Pattern 6: Inconsistent schema-to-type derivation patterns (z.infer vs explicit interface)

- **Seen in**: packages-contracts-src-contracts-10 (LinkedIn uses z.infer, Microsoft Calendar uses z.infer, Linear uses explicit interface, Jira uses explicit interface)
- **Category**: missing-strict-typing
- **Combined impact**: low -- cosmetic inconsistency with a TS7056 risk as schemas grow
- **What's happening**: Some platform expanded-data schemas use the explicit-interface pattern (`export interface FooExpandedData { ... }` with `z.ZodType<FooExpandedData>` annotation) while others use `z.infer<typeof schema>` directly. The explicit-interface pattern is the documented standard and prevents TS7056 "excessively deep type instantiation" errors on complex schemas.
- **Suggestion**: Convert the remaining `z.infer` usages (LinkedIn, Microsoft Calendar) to the explicit-interface pattern for consistency. This is low-priority since the affected schemas are currently small.
- **Estimated scope**: 2-4 files

## Pattern 7: AI SDK type mismatches worked around with double casts

- **Seen in**: apps-app-src-sentry.ts (double cast on PerformanceObserver entries), apps-app-src-utils-1 (sanitizeMessagesForChatRequest casts message to access .parts, then casts result to UIMessage["parts"], isToolPart casts to access .type and .toolCallId)
- **Category**: type-cast
- **Combined impact**: medium -- concentrated in AI/SDK boundary code but involves fragile multi-step casts
- **What's happening**: At SDK interop boundaries (AI SDK, PerformanceObserver API), values are cast through intermediate types to access fields that either (a) exist on the real type but are not exposed in the SDK's type definitions, or (b) differ between SDK versions. The double-cast pattern (`value as IntermediateType` then `result as FinalType`) is fragile because neither cast is validated, and SDK upgrades can silently break the intermediate type shape.
- **Suggestion**: For AI SDK: verify that the installed SDK version's `UIMessage` type includes `parts` and `type` discriminants directly. If so, remove the intermediate casts. If the SDK types lag behind the runtime, file an issue upstream and add a comment with the SDK version gap. For PerformanceObserver: remove the second cast and access `PerformanceEventTiming` members directly.
- **Estimated scope**: 2-3 files, 4-5 cast sites
