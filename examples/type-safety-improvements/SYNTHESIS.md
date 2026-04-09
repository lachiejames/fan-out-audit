# Type-Safety Audit Synthesis

**Task**: Find type-safety improvement opportunities across apps/api/src, apps/app/src, packages/contracts/src, packages/ui/src
**Date**: 2026-04-09
**Slices audited**: 249
**Total files inspected**: ~1834
**Phase 2 groups**: 21

---

## Top 10 Highest-Impact Opportunities

Ranked by impact x prevalence. Each item cites specific Phase 1/Phase 2 files as evidence, includes file paths and line numbers where available, and gives a concrete actionable suggestion.

### 1. Untyped and weakly-typed JSONB columns in Drizzle table definitions

- **Impact**: high -- this is the single largest type-safety gap in the backend; every untyped jsonb column forces downstream code to cast or assert, creating a cascade of secondary `as` casts throughout the codebase
- **Category**: missing-strict-typing / record-weakening
- **Evidence**: Found across 13 Phase 2 groups (G1, G2, G3, G4, G5, G6, G7, G8, G9, G10, G11, G12, G18). Phase 1 slices `apps-api-src-shared-3` (Finding 1: 15 identical cursor aliases all equal `Record<string, unknown>`; Findings 2-3: `payload` columns without `$type<>`), `apps-api-src-shared-4` (gold-standard counterexample: `chat-messages.table.ts` uses `.$type<UiPart[]>()`). At least 30 jsonb columns across 15+ table files in `apps/api/src/shared/db/tables/` lack `.$type<>()` annotations or use `.$type<Record<string, unknown>>()`.
- **What to do**: Systematically add `.$type<T>()` to every jsonb column. For columns with a discriminant column on the same table (e.g., `platform` + `cursorState`, `eventType` + `payload`), use discriminated unions. For columns that genuinely store open-ended data, define a partial interface with known fields plus an index signature. The gold-standard pattern from `chat-messages.table.ts` (`.$type<UiPart[]>()` imported from contracts) is the template. Start with the highest-traffic columns: `predictions.predictedActions`, `suggestions.compoundSuggestion`, `sync-checkpoints.cursorState`, `domain-events.payload`, `knowledge-source-revisions` (8 untyped jsonb columns).
- **Estimated scope**: ~30 jsonb columns across ~15 table files, eliminating 100+ downstream `as` casts

### 2. Duplicate type definitions that already exist in `@slopweaver/contracts`

- **Impact**: high -- when contract schemas evolve, local copies silently drift. Already demonstrated: `FetchLinearIssueResult` drops fields that `LinearIssueResult` provides (Phase 1 `apps-api-src-application-1` Finding 1), `ResponseTimeByHour` diverges between local and contract (G4 Pattern 1)
- **Category**: duplicate-type / sdk-type-duplication
- **Evidence**: Found in 19 of 21 Phase 2 groups. Phase 1: `apps-api-src-application-1` Finding 1 (`FetchLinearIssueResult`), `apps-app-src-shared-app-1` Finding 2 (`MessagePriority`), `packages-contracts-src-contracts-1` Findings 4-5 (`AnalyticsResponse`, `WorkItemRevision` aliases). G1 Pattern 5 lists 7 slices with duplicates; G2 Pattern 1 lists 8 slices; G3 Pattern 2 lists 8 slices with 25+ local type definitions; G15 Pattern 3 lists 20+ local types across 10+ hook files. The `BackendIntegration` interface alone is defined in 3 separate files (G15 Pattern 4). `SubscriptionStatus` literal union appears 11+ times inline (G5 Pattern 2). `LoggerLike` is duplicated 4 times in the chat module (G10 Pattern 3). `RawBodyRequest` is copy-pasted in 5 webhook guard files (G9 Pattern 6). Digest types duplicated in 3 locations (G1 Pattern 5). Five identical `*ParseResult` interfaces across parser files (G1 Pattern 5).
- **What to do**: Adopt a "derive, don't duplicate" rule. Every service-layer type that represents a contract response or DB row should be derived from the canonical source (`z.infer<typeof schema>`, `typeof table.$inferSelect`, or import from `@slopweaver/contracts`). Immediate actions: (a) delete `BackendIntegration` from 3 hook files and create one canonical export; (b) replace 11+ inline `SubscriptionStatus` unions with the contracts import; (c) extract `LoggerLike` to a shared location; (d) extract `RawBodyRequest` to `apps/api/src/interface/webhooks/types/`; (e) sweep all `apps/app/src/hooks/` for local interfaces that match contract response types.
- **Estimated scope**: 80+ duplicate type definitions across 50+ files

### 3. `Record<string, T>` lookup maps where keys are a known finite union

- **Impact**: high -- the single most pervasive type-safety gap in both frontend and backend; defeats exhaustiveness checking, allows typos, and hides missing-key bugs
- **Category**: record-weakening
- **Evidence**: Found in 20 of 21 Phase 2 groups. Phase 1: `packages-contracts-src-contracts-1` Finding 1 (`MODEL_TIER_LABELS: Record<string, string>`), `apps-app-src-shared-app-1` Finding 5 (`PLATFORMS[platformId as PlatformId]`). G13 Pattern 1 lists 15+ constant maps in frontend components; G14 Pattern 1 lists 15-20 maps across 12-15 files; G16 Pattern 1 lists 15-20 files with 20-25 instances. Backend examples: `SUGGESTION_PREFIX_MAP`, `CATEGORY_LABELS`, `TODO_PRIORITY_MAP`, `MIME_TO_PROVIDER`, `STT_MIME_TO_EXTENSION`, `endpointClasses`, `OUTBOUND_RATE_LIMITS` (G3 Pattern 3, G11 Pattern 4). UI: `PLATFORM_STYLES`, `PLATFORM_ICONS`, `CONTEXT_BADGE_COLORS` (G19 Pattern 5).
- **What to do**: Systematically replace `Record<string, T>` with `Record<UnionType, T>` (when exhaustive) or `Partial<Record<UnionType, T>>` (when subset). For constant objects, remove the explicit `Record<string, T>` annotation and use `as const satisfies Record<string, T>` to preserve narrow key inference. A codebase-wide grep for `Record<string,` combined with an ESLint rule would prevent regression. Prioritize frontend lookup maps (icons, labels, colors, badges) since they directly affect rendering correctness.
- **Estimated scope**: 60+ constant declarations across 40+ files

### 4. `BaseSyncService` uses `unknown[]` for parsed items, forcing casts in every platform subclass

- **Impact**: high -- affects every sync service integration (10-15 subclasses), each with 2-3 cast sites. This is a single root-cause fix with maximum leverage.
- **Category**: type-cast
- **Evidence**: Found in 4 Phase 2 groups: G6 Pattern 1 (Asana, GitHub, Google Docs, Google Drive, Google Gmail), G7 Pattern 1 (Jira, Linear, OneDrive, Monday), G8 Pattern 1 (Notion, Slack), G8 Pattern 7 (identical copy-paste pattern). Phase 1: `apps-api-src-integrations-19` (Gmail `parsedItems as ParsedGmailMessage[]`). Every platform sync service immediately casts `parsedItems as PlatformSpecificType[]` in `getEmbeddingTexts`, `syncAttachments`, and `onContentInserted` overrides.
- **What to do**: Add a generic type parameter to `BaseSyncService`: `class BaseSyncService<..., TParsedItem>`. Override signatures become `getEmbeddingTexts(parsedItems: TParsedItem[]): string[]`. Each subclass parameterizes with its concrete type (e.g., `JiraSyncService extends BaseSyncService<..., ResolvedIssue>`). This eliminates all downstream casts in one change. File: `apps/api/src/integrations/core/abstractions/base-sync.service.ts`.
- **Estimated scope**: 1 base class change + 10-15 sync service files, eliminating 30-45 casts

### 5. `integration.platformMetadata` typed `unknown` forces repeated casts in every OAuth/plugin/sync service

- **Impact**: high -- at least 20 distinct cast sites across all platform integrations, each inventing its own inline cast shape
- **Category**: type-cast
- **Evidence**: Found in G6 Pattern 2 (8 platforms listed), G7 Pattern 5 (Google, Jira, LinkedIn, Outlook, Monday), G8 Pattern 2 (Notion, Slack). Phase 1: `apps-api-src-integrations-12` Findings 1, 4 (Facebook Messenger casts `platformMetadata as { pageId?: string } | null` and `as FacebookMessengerPlatformMetadata | null`). Each service invents its own cast shape: `Partial<AsanaPlatformMetadata>`, `GoogleCalendarPlatformMetadata | null`, `SlackPlatformMetadata | null`, `{ pageId?: string } | null`, etc.
- **What to do**: For each platform, define a Zod schema matching the platform metadata interface. Add a shared utility `parsePlatformMetadata<T>({ raw, schema }: { raw: unknown; schema: z.ZodType<T> }): T | null` that validates before returning. Call it once at the service entry point. Consider adding a generic parameter to the OAuth base service so `this.integration.platformMetadata` returns the correct platform type automatically.
- **Estimated scope**: ~12 platform metadata Zod schemas to create + ~20 cast sites to replace

### 6. `platform: string` parameters and fields where `PlatformId` union exists

- **Impact**: high -- `PlatformId` is the canonical type but many service methods, component props, and contract schemas accept raw `string`
- **Category**: missing-strict-typing
- **Evidence**: Found in 14 Phase 2 groups. Phase 1: `apps-api-src-application-1` Finding 4 (`resolveIntegrationId({ platform: string })`), `packages-contracts-src-contracts-1` Findings 2-3 (`agentToolMetadataSchema.platforms: z.array(z.string())`), `apps-app-src-shared-app-1` Finding 5. G17 Pattern 2 lists 8-10 files. G2 Pattern 6 lists 6-10 files. G13 Pattern 4, G14 Pattern 2, G15 Pattern 2, G21 Pattern 4 all flag this in frontend components. Analytics event properties in contracts also use bare `z.string()` for platform fields (G17 Pattern 2).
- **What to do**: Replace `platform: string` with `platform: PlatformId` at every internal boundary. In Zod schemas, use `z.string().refine(isPlatformId)` or `z.enum([...PLATFORM_IDS])`. Keep `string` only at external API controller boundaries where Zod validates. For frontend component props, use `PlatformId | (string & {})` to preserve autocomplete while accepting arbitrary strings. Ensure `isPlatformId` is declared as a proper type predicate (`value is PlatformId`).
- **Estimated scope**: 40+ parameter/field sites across 25+ files in api, app, and contracts

### 7. Unsafe `as SomeType` casts on JSONB/`unknown` data without runtime validation

- **Impact**: high -- at least 50 cast sites across the codebase where external data (DB JSONB, HTTP responses, job payloads, pub/sub events) is cast to typed interfaces without Zod validation
- **Category**: type-cast
- **Evidence**: G2 Pattern 4 (8 files, ~30 casts on JSONB/raw-SQL), G3 Pattern 4 (~40+ casts including 30+ `as any` in test fixtures), G4 Pattern 3 (8 instances on HTTP responses/JWT payloads), G7 Pattern 7 (4 API clients casting `response.json() as T`), G9 Pattern 8 (SSE endpoints casting `req.body as { ... }`). Phase 1: `apps-api-src-integrations-12` Finding 2 (`page_id`/`page_name` smuggled through `as Record<string, unknown>` then `as string`), Finding 5 (`response.json() as { data: Array<Record<string, unknown>> }`). `apps-api-src-application-10` Finding 2 (`metadata["subject"] as string | undefined`).
- **What to do**: Introduce validation at trust boundaries. (1) For JSONB columns, fix at the table level with `.$type<T>()` (see Opportunity 1). (2) For HTTP response bodies, add a `schema: z.ZodType<T>` parameter to platform API client `get<T>`/`post<T>` methods. (3) For BullMQ job payloads, parse through the corresponding contract Zod schema. (4) For SSE endpoints that bypass ts-rest, add explicit `bodySchema.parse(req.body)`. A shared `parsePayload({ schema, data })` utility would standardize the approach.
- **Estimated scope**: 50+ cast sites across 30+ files

### 8. Copy-pasted `result as { usage?: unknown }` on Vercel AI SDK `generateText` results

- **Impact**: medium -- 8-10 identical code blocks across 8+ services, all doing unnecessary work that discards the SDK's precise `LanguageModelUsage` type
- **Category**: type-cast
- **Evidence**: G1 Pattern 3 (3 files), G2 Pattern 7 (2 files), G3 Pattern 5 (3 files), G5 Pattern 1 (8 instances across 6 files), G8 Pattern 7 (2 files). Phase 1: `apps-api-src-integrations-6` Findings 5-6 (`query-expansion.service.ts`, `reply-classifier.service.ts`), `apps-api-src-application-10` Finding 1 (`auto-compaction.service.ts`).
- **What to do**: The root cause is that `extractInputOutputTokensFromUsage` accepts `unknown` for its `usage` parameter. Fix this single utility to accept `LanguageModelUsage` (from the `ai` package). Then replace all 8-10 guard+cast blocks with `const usage = result.usage;`. Also extract `extractVoyageTokenUsage` for Voyage API patterns (G5 Pattern 1).
- **Estimated scope**: 1 utility signature change + 8-10 files with identical 3-line block removal

### 9. Error body casts on ts-rest non-200 responses (`response.body as { message?: string }`)

- **Impact**: high -- at least 20 instances across hooks, pages, and voice modules, with inconsistent field access (`message` vs `error` vs `details`)
- **Category**: type-cast
- **Evidence**: G14 Pattern 3 (5+ files in voice module), G15 Pattern 1 (13 instances across 8 files), G16 Pattern 2 (15-18 instances across 10-12 files). The field names are inconsistent: some check `message`, some check `error`, some check `message | string[]`. Meanwhile, `extractErrorMessage` from `@slopweaver/contracts` already handles all variants safely.
- **What to do**: Replace every `response.body as { message?: ... }` pattern with `extractErrorMessage(response.body)` from `@slopweaver/contracts`. A codemod searching for `as { message` and `as { error` in `apps/app/src/` would catch most instances. For the voice module specifically, also create an `extractApiErrorBody` type guard utility.
- **Estimated scope**: 20+ cast sites across 15+ files in `apps/app/src/`

### 10. `z.record(z.string(), z.unknown())` as the default metadata/payload shape in contracts

- **Impact**: high -- this is the root cause that propagates `Record<string, unknown>` to every consumer of contracts-defined types
- **Category**: record-weakening / any-usage
- **Evidence**: G18 Pattern 1 (found in 8 of 12 contract slices). Key schemas: `actionTransactionSchema.metadata`, `baseContentSchema.metadata`, `integrationActionPayloadSchema`, `captureMetadata`, `knowledgeSourceItemResponseSchema.metadata`, `createMessageBodySchema.platformIdentifiers`, `baseWorkItemSchema.aiSources`, `selectedScopeSchema.params`. Phase 1: `packages-contracts-src-contracts-1` (contracts infrastructure).
- **What to do**: For fields where the shape varies by discriminant (platform, action type), define discriminated union sub-schemas. For fields with a single known shape, define an explicit object schema. For genuinely dynamic fields (Asana custom fields, Monday column values), narrow the value type to `z.union([z.string(), z.number(), z.boolean(), z.null()])`. Start with the highest-traffic paths: `integrationActionPayloadSchema`, `baseContentSchema.metadata`, and `baseWorkItemSchema.aiSources`.
- **Estimated scope**: 15-20 schema fields across 10+ contract files

---

## Cross-Cutting Patterns (deduplicated from Phase 2)

### Record Weakening: `Record<string, unknown>` metadata on domain objects

- **Seen in groups**: G1, G2, G3, G4, G5, G6, G7, G8, G9, G10, G11, G12, G13, G14, G15, G16, G17, G18, G19, G20, G21
- **Combined impact**: high
- **Description**: The most pervasive pattern across the entire codebase. `Record<string, unknown>` is used for metadata, cursor state, event payloads, lookup maps, Drizzle `.set()` accumulators, and API request bodies. Every instance forces downstream consumers to cast individual fields, creating a cascade of secondary `as` casts. Three distinct sub-patterns: (a) JSONB columns inferred as `unknown`; (b) in-memory DTOs with known field shapes; (c) lookup maps with finite key sets.
- **Fix**: Three-tier approach. (1) JSONB columns: add `.$type<T>()`. (2) DTOs: define discriminated unions per `itemType`/`platform` discriminant. (3) Lookup maps: use `Record<UnionKey, T>` or `as const satisfies`.

### Record Weakening: `Record<string, unknown>` as Drizzle `.set()` accumulator

- **Seen in groups**: G2, G3
- **Combined impact**: high
- **Description**: At least 10 services build `Record<string, unknown>` objects and pass them to Drizzle `.set()`, bypassing column-name and value-type checking. A misspelled column name compiles silently.
- **Fix**: Replace with `Partial<typeof table.$inferInsert>`. Use named property assignments instead of bracket notation.

### Type Cast: `as SomeType` on unvalidated `unknown` payloads

- **Seen in groups**: G1, G2, G3, G4, G5, G6, G7, G8, G9, G10, G13, G14, G15, G16, G17
- **Combined impact**: high
- **Description**: The second most common pattern. Appears in three recurring contexts: (a) BullMQ job payload casts; (b) knowledge-source parser casts on raw JSON; (c) Drizzle JSONB results cast to application types. In all cases, Zod schemas either already exist or could be trivially created.
- **Fix**: Validate at trust boundaries with Zod `.safeParse()`. Create a shared `parsePayload({ schema, data })` utility.

### Type Cast: `BaseSyncService` / `fetchThread` forcing `unknown[]` / `Promise<unknown>` returns

- **Seen in groups**: G6, G7, G8
- **Combined impact**: high
- **Description**: `BaseSyncService` declares `parsedItems: unknown[]` and `fetchThread` returns `Promise<unknown>`. Every platform subclass must immediately cast. This is a single root cause with 30+ downstream casts.
- **Fix**: Add generic type parameters to `BaseSyncService<TParsedItem>` and per-platform thread result interfaces.

### Type Cast: `platformMetadata as PlatformMetadataType | null` from DB `unknown`

- **Seen in groups**: G6, G7, G8
- **Combined impact**: high
- **Description**: The `integrations` table stores `platformMetadata` as untyped JSONB. Every OAuth service casts it without validation. Each platform invents its own cast shape.
- **Fix**: Per-platform Zod schemas + shared `parsePlatformMetadata` utility.

### Type Cast: `filter(Boolean) as T[]` instead of type-predicate filter

- **Seen in groups**: G1, G6, G7
- **Combined impact**: medium
- **Description**: After `.map()` returning `T | undefined`, `.filter(Boolean) as T[]` is used because TypeScript cannot narrow through `Boolean`. At least 8-10 instances.
- **Fix**: Replace with `.filter((x): x is T => x != null)` or create a shared `isDefined<T>` utility.

### Type Cast: Copy-pasted AI SDK `usage` extraction with unnecessary guards

- **Seen in groups**: G1, G2, G3, G5, G8
- **Combined impact**: medium
- **Description**: 8-10 identical code blocks cast `generateText` result to `{ usage?: unknown }` despite the SDK already typing `usage` as `LanguageModelUsage`.
- **Fix**: Fix `extractInputOutputTokensFromUsage` to accept `LanguageModelUsage`, then use `result.usage` directly.

### Duplicate Type: Hand-rolled interfaces mirroring contract/Zod-inferred types

- **Seen in groups**: G1, G2, G3, G4, G5, G7, G9, G10, G12, G13, G14, G15, G16, G17, G18, G20
- **Combined impact**: high
- **Description**: 80+ local type definitions across the codebase duplicate types from `@slopweaver/contracts`. The frontend hooks layer alone has 20+ (G15 Pattern 3). Active drift has been demonstrated (G4 Pattern 1: `ResponseTimeByHour` divergence).
- **Fix**: "Derive, don't duplicate" rule. Import from contracts or derive with `z.infer<>`, `Pick`, `Omit`.

### Duplicate Type: Inline string-literal unions duplicating DB enums or contract Zod enums

- **Seen in groups**: G1, G2, G3, G5, G9, G12
- **Combined impact**: high
- **Description**: `SubscriptionStatus` appears 11+ times inline. `ArchiveStatus`, `ProactiveTriggerType`, `PushChannel`, and others are duplicated similarly. Adding a variant requires touching all copies.
- **Fix**: Import the canonical type from `@slopweaver/contracts` or `enums.ts`.

### Missing Strict Typing: `z.string()` for timestamps instead of `isoDateTimeStringSchema`

- **Seen in groups**: G18, G19
- **Combined impact**: medium
- **Description**: 20-25 timestamp fields across platform schemas use bare `z.string()` instead of the codebase's `isoDateTimeStringSchema`.
- **Fix**: Bulk grep for timestamp field names in contracts and replace with `isoDateTimeStringSchema`.

### Missing Strict Typing: `z.string()` for email fields instead of `z.email()`

- **Seen in groups**: G18, G19
- **Combined impact**: low-medium
- **Description**: 6-8 email fields across platform metadata schemas lack format validation.
- **Fix**: Replace `email: z.string()` with `email: z.email()`.

### Missing Strict Typing: Positional parameters on exported functions violating named-params rule

- **Seen in groups**: G4, G5
- **Combined impact**: medium
- **Description**: At least 10 abstract methods across 6 domain port files use positional params. Several have two same-typed `string` params that can be silently swapped.
- **Fix**: Mechanical refactor to named-object params in `apps/api/src/domain/ports/services/`.

### Any Usage: `any[]` in NestJS queue/module factory patterns

- **Seen in groups**: G5
- **Combined impact**: medium
- **Description**: `createQueueService` returns `new (...args: any[])` and `createQueueModule` uses `any[]`. This propagates through all ~10 queue modules.
- **Fix**: Narrow constructor signatures and use NestJS `Type` types.

### Any Usage: Mutation hooks returning `Promise<unknown>` instead of typed response

- **Seen in groups**: G15
- **Combined impact**: high
- **Description**: 15+ mutation return types in frontend hooks lose their contract-derived type.
- **Fix**: Derive explicit return types from contract response schemas.

---

## Quick Wins (low effort, high value)

1. **Remove redundant `as Date` casts on `calculateTokenExpiry`**: `apps/api/src/integrations/platforms/google/utils/google-oauth.utils.ts` and `apps/api/src/integrations/platforms/linear/utils/linear-oauth.utils.ts` -- remove `as Date` since the function already returns `Date`. (G7 Pattern 8, 2 files, 1 line each)

2. **Extract `RawBodyRequest` to shared file**: `apps/api/src/interface/webhooks/types/raw-body-request.ts` -- move the 3-line interface from 5 webhook guard files to one shared location. (G9 Pattern 6, 5 files)

3. **Replace `filter(Boolean) as T[]` with typed filter predicate**: Create a shared `isDefined<T>(x: T | undefined): x is T` utility and use `.filter(isDefined)` across all sync services. (G6 Pattern 4, G7 Pattern 2, ~8 instances)

4. **Fix `extractInputOutputTokensFromUsage` signature**: Change parameter type from `{ usage: unknown }` to `{ usage: LanguageModelUsage }`, then delete 8-10 identical guard+cast blocks. (G1 Pattern 3, G5 Pattern 1)

5. **Remove `extends Record<string, unknown>` from Microsoft platform metadata interfaces**: `microsoft-onedrive-oauth.service.ts`, `microsoft-outlook-oauth.service.ts`, `microsoft-teams-oauth.service.ts` -- remove redundant index signatures. (G7 Pattern 6, 3 files)

6. **Extract `LoggerLike` to shared location**: Move from 4 copies in `apps/api/src/interface/http/chat/` to `apps/api/src/shared/types/logger-like.ts`. (G10 Pattern 3, 4 files)

7. **Fix `isPlatformId` type predicate**: Ensure it returns `value is PlatformId` not `boolean`, which eliminates redundant `as PlatformId` casts across the entire codebase. (G21 Pattern 2, G13 Pattern 5)

8. **Replace `z.string()` timestamps with `isoDateTimeStringSchema`**: Grep for `(createdAt|updatedAt|timestamp|submittedAt|created_time):\s*z\.string\(\)` in `packages/contracts/src/` and replace. (G18 Pattern 2, G19 Pattern 1, ~20 fields)

9. **Add Window interface augmentation for debug properties**: Create `apps/app/src/types/globals.d.ts` declaring all `__SLOPWEAVER_*` properties to eliminate 5+ inline window casts. (G14 Pattern 6, G16 Pattern 6)

10. **Replace `MODEL_TIER_LABELS` and `TASK_TYPE_LABELS` key types**: In `packages/contracts/src/contracts/ai-costs/constants.ts:28-46`, change `Record<string, string>` to `Record<(typeof ALL_MODEL_TIERS)[number], string>` and `Record<(typeof ALL_TASK_TYPES)[number], string>`. (Phase 1 `packages-contracts-src-contracts-1` Finding 1)

---

## Slices with no findings

The following slices reported zero findings and appear genuinely clean:

- **packages-ui-src-atoms-5** (8 Radix UI primitive wrappers using standard `ComponentProps` pattern)
- **packages-ui-src-atoms-6** (8 atoms, all clean)
- **packages-ui-src-atoms-7** (clean atoms)
- **packages-ui-src-atoms-9** (clean atoms)
- **packages-ui-src-molecules-5** (clean molecules)
- **packages-ui-src-molecules-6** (clean molecules)
- **packages-ui-src-index.ts** (barrel export, no logic)
- **apps-app-src-env.ts** (single env file, clean)
- **apps-app-src-lib-7** (0 findings)
- **apps-api-src-seed-data-2** (auto-generated seed data, 0 findings)
- **apps-api-src-interface-22** (0 findings)

**Assessment**: These are genuinely clean. The UI package atoms and molecules that wrap Radix UI primitives consistently use the `React.ComponentProps<typeof Primitive>` pattern, which is inherently type-safe. The seed-data-2 slice is auto-generated. The env.ts file is minimal. This confirms that remediation effort should focus on the application layer, integrations layer, hooks layer, and Drizzle table definitions -- not the UI component library.

Additionally, Group 20 Pattern 5 explicitly notes that 5 of 12 UI slices had zero findings, confirming the UI atoms layer is well-typed as a whole.

---

## Raw data

- Phase 1 (per-slice): `docs/audits/type-safety-improvements/phase1/`
- Phase 2 (cross-cutting): `docs/audits/type-safety-improvements/phase2/`
- Slice plan: `docs/audits/type-safety-improvements/SLICE_PLAN.md`
- Task: `docs/audits/type-safety-improvements/TASK.md`
