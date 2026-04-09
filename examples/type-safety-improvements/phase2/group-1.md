# Cross-Cutting Patterns (Group 1)

**Slices analyzed**: apps-api-src-application-1, apps-api-src-application-2, apps-api-src-application-10, apps-api-src-application-11, apps-api-src-application-12, apps-api-src-application-13, apps-api-src-application-14, apps-api-src-application-15, apps-api-src-application-16, apps-api-src-application-17, apps-api-src-application-18, apps-api-src-application-19

## Pattern 1: `Record<string, unknown>` used for metadata/details/context objects with known shapes

- **Seen in**: slice-1 (FetchContentError.details), slice-2 (ListTodosResultWithContext.query.filters), slice-10 (EnrichedContext.metadata, SearchResult.metadata), slice-11 (ParsedWorkspaceUrl.identifiers), slice-12 (FilterableQuery fields all typed unknown), slice-13 (platformIdentifiers cast to Record), slice-14 (integrationIdMap dual-key Record), slice-17 (DriveSnapshotManifest, ChatGPTNode.metadata), slice-18 (NormalizedSourceItem.metadata), slice-19 (MIME_TO_PROVIDER, checkpoint/metadata JSONB columns)
- **Category**: record-weakening
- **Combined impact**: high -- this is the single most pervasive type-safety gap across the application layer. Every instance forces downstream consumers to re-cast or runtime-guard individual fields, creating a cascade of secondary `as` casts throughout the codebase.
- **What's happening**: When a function returns or accepts data with a known set of keys, the code types it as `Record<string, unknown>` instead of an interface or discriminated union. This happens in three distinct contexts: (a) in-memory DTOs like `EnrichedContext.metadata`, `NormalizedSourceItem.metadata`, and `ParsedWorkspaceUrl.identifiers` where the keys are fixed per discriminant value; (b) Drizzle JSONB columns like `checkpointJson`, `metadataJson`, and `payload` that are inferred as `unknown` from the ORM; (c) lookup maps like `MIME_TO_PROVIDER` and `integrationIdMap` where the key set is finite. In all three cases, callers must use bracket-notation string access and manual typeof guards, which the compiler cannot validate.
- **Suggestion**: Fix in three tiers. (1) For JSONB columns, add `.$type<T>()` to the Drizzle column definitions -- this is a one-line fix per column that eliminates all downstream casts for that column simultaneously. (2) For DTOs with a discriminant field (like `itemType` or `platform`), convert to discriminated unions so TypeScript narrows the metadata shape automatically. (3) For lookup maps, remove the explicit `Record<string, T>` annotation and use `as const satisfies` or `Partial<Record<KnownKey, T>>` so undefined-on-miss is visible.
- **Estimated scope**: 20+ files, 30+ individual Record usages

## Pattern 2: `as SpecificType` casts on unvalidated `unknown` / loosely-typed payloads

- **Seen in**: slice-2 (as Record<string, unknown> on settings updates), slice-10 (as { usage?: unknown } on AI SDK result), slice-11 (as Record<string, unknown> in asRecord helper), slice-12 (as ContentResponse["metadata"] repeated twice, as Record<string, unknown> in asRecord), slice-13 (as Record<string, unknown> on platformIdentifiers, as ConversationSourceAnchor["itemType"]), slice-14 (as typeof ctx on parsed JSON, as string[] after filter(Boolean)), slice-15 (as ContentActionTakenPayload on job.payload), slice-16 (as OAuthCallbackCompletedPayload / SyncContentIndexedPayload / WebhookProcessingFailedPayload on job.payload), slice-17 (as ChatGPTConversation after typeof check), slice-18 (as ClaudeConversation / ClaudeMemory / ClaudeProject after typeof check, as unknown[] on array value), slice-19 (as Record<string, unknown> on JSONB columns, as KnowledgeSourceProvider on unvalidated string)
- **Category**: type-cast
- **Combined impact**: high -- these casts suppress compile-time errors for data that has not been structurally validated. Each one is a potential runtime crash when the actual shape diverges from the asserted type.
- **What's happening**: The codebase has a consistent anti-pattern: check `typeof x === "object" && x !== null`, then immediately `as SpecificInterface`. This proves the value is an object but verifies zero properties. The pattern appears in three recurring contexts: (a) domain event handlers casting `job.payload` (4 handlers across slices 15-16); (b) knowledge-source parsers casting raw JSON array elements to parser-specific interfaces (slices 17-18); (c) Drizzle JSONB results cast to application types (slice 19). In all cases, Zod schemas for the target types either already exist in `@slopweaver/contracts` or could be trivially created from the existing interfaces.
- **Suggestion**: Introduce a validation boundary at each cast site. For domain event handlers, parse `job.payload` through the corresponding contract Zod schema (the schemas already exist). For knowledge-source parsers, create lightweight Zod schemas or type-guard functions for the parser-specific interfaces. For Drizzle JSONB, type the column at the schema level (see Pattern 1). A shared utility like `parsePayload({ schema, data })` wrapping `schema.safeParse` would standardize the approach.
- **Estimated scope**: 15+ files, 20+ cast sites

## Pattern 3: Unnecessary type guard + unsafe cast on Vercel AI SDK `generateText` result.usage

- **Seen in**: slice-10 (auto-compaction.service.ts), slice-13 (conversation-title-generation.service.ts), slice-16 (writing-pattern-extraction.service.ts)
- **Category**: type-cast
- **Combined impact**: medium -- three identical code blocks across three services, all doing unnecessary work that obscures the SDK's actual type.
- **What's happening**: After calling `generateText(...)` from the Vercel AI SDK, all three services perform the same defensive pattern: `typeof result === "object" && result !== null && "usage" in result ? (result as { usage?: unknown }).usage : undefined`. The `generateText` return type (`GenerateTextResult`) already has `readonly usage: LanguageModelUsage` -- the runtime check is dead code, the cast throws away the typed `LanguageModelUsage` object, and `extractInputOutputTokensFromUsage` then has to re-narrow from `unknown`. This pattern was likely copy-pasted from a single source.
- **Suggestion**: In all three files, replace the guard+cast block with `const usage = result.usage;` (one line). If `extractInputOutputTokensFromUsage` currently accepts `{ usage: unknown }`, tighten its parameter to `{ usage: LanguageModelUsage }` (imported from `"ai"`) so the type flows end-to-end.
- **Estimated scope**: 3 files, 3 instances (identical fix in each)

## Pattern 4: `platform` / `source` parameters typed as `string` instead of a constrained union

- **Seen in**: slice-1 (resolveIntegrationId platform: string in fetch-content.use-case.ts), slice-2 (resolveIntegrationId platform: string in manage-projects.use-case.ts, ManageProjectsError.platform: string), slice-12 (source: string in content-metadata.utils.ts, content-preview.utils.ts), slice-19 (providerHint: string in knowledge-source-preflight.service.ts)
- **Category**: missing-strict-typing
- **Combined impact**: medium -- string-typed platform/source parameters accept any arbitrary value without compile-time feedback. Typos like `"google_gmail"` or `"slacc"` compile silently and produce confusing runtime errors.
- **What's happening**: Multiple files accept platform identifiers or content sources as bare `string` even though constrained union types already exist in `@slopweaver/contracts` (`PlatformId`, `ProjectManagementPlatform`, `ContentSource`). All callers pass literal values from these unions, but the function signatures don't enforce it. This appears to stem from early development where the union types may not have existed yet, or from a desire to avoid import dependencies.
- **Suggestion**: Replace `platform: string` with `platform: PlatformId` (or the appropriate narrower union) and `source: string` with `source: PlatformId | BuiltinContentSource`. The contracts package already exports these types. Each fix is a one-line parameter type change plus an import.
- **Estimated scope**: 6+ files, 8+ parameter sites

## Pattern 5: Duplicate type definitions that already exist in `@slopweaver/contracts` or sibling modules

- **Seen in**: slice-1 (FetchLinearIssueResult duplicates LinearIssueResult), slice-11 (CompactionResult duplicated in service and utils), slice-12 (FilterableQuery duplicates ListContentQuery fields, inline logger duplicates ILoggerPort), slice-13 (DemoGithub* types duplicate GithubExpandedData nested types), slice-14 (inline ctx shape duplicates ProfileCurrentContext, map callback annotations duplicate ProfileContentFields), slice-15 (DigestUrgentItem/DigestStats/DigestInsights/DigestResponse duplicated in service AND utils -- already in contracts), slice-18 (five identical *ParseResult interfaces across parser files)
- **Category**: duplicate-type
- **Combined impact**: high -- when contract types evolve, local copies silently drift. This has already caused data truncation (slice-1: FetchLinearIssueResult drops fields that LinearIssueResult provides). The digest types (slice-15) are the worst case: four types duplicated in three locations (contracts, service, utils).
- **Suggestion**: For each duplicate: (a) if the type exists in `@slopweaver/contracts`, delete the local copy and import from contracts; (b) if two local files define the same type, consolidate into one canonical location and import; (c) for the five parser result interfaces (slice-18), define a single `KnowledgeSourceParseResult` in `knowledge-source-parser.types.ts`. The digest fix (slice-15) alone removes 8 redundant interface declarations.
- **Estimated scope**: 12+ files, 15+ duplicate type declarations

## Pattern 6: `eventType` / `rating` / `sourceType` fields typed as `string` instead of known literal unions

- **Seen in**: slice-1 (sourceType: "integration_message" as const -- 3 instances), slice-10 (FeedbackResponse.rating: string -- should be "positive" | "negative"), slice-14 (PendingUpdate.eventType: string -- should be 3-member union), slice-15 (entity-resolution signals.type: string then cast back to 4-member union), slice-19 (getEventHistory eventType: string -- should be ImportProgressPayload union)
- **Category**: missing-strict-typing
- **Combined impact**: medium -- bare `string` fields that only ever hold a small set of known values prevent exhaustiveness checking and IDE autocompletion. In slice-15, the wide `string` type on signals.type directly causes a downstream `as` cast to re-narrow it, creating a type-cast problem that would not exist if the field were properly typed from the start.
- **Suggestion**: For each field, extract a string literal union type and apply it at the declaration site. This eliminates downstream `as const` casts (slice-1) and `as UnionType` re-narrowings (slice-15) as a side effect. Most of these unions have 2-5 members and are trivially derivable from usage-site grep.
- **Estimated scope**: 8+ files, 10+ field declarations

## Pattern 7: `filter(Boolean)` and `.find()` results losing type narrowing, patched with casts or `!`

- **Seen in**: slice-1 (Array.find()! non-null assertions on formatToParts -- 3 instances), slice-14 (filter(Boolean) then as string[]), slice-17 (conversation.mapping! non-null assertion)
- **Category**: type-cast / missing-strict-typing
- **Combined impact**: medium -- these are common TypeScript ergonomic gaps. `filter(Boolean)` does not narrow `(T | null)[]` to `T[]`, and `.find()` returns `T | undefined`. Papering over with `!` or `as T[]` silently converts potential runtime nulls into crashes.
- **Suggestion**: Replace `filter(Boolean) as T[]` with `.filter((x): x is T => x !== null)`. Replace `array.find(pred)!` with either optional chaining + fallback, or a guard that throws a descriptive error. These are mechanical, low-risk substitutions. Consider adding an ESLint rule (`@typescript-eslint/no-non-null-assertion`) to catch future instances.
- **Estimated scope**: 5+ files, 8+ instances
