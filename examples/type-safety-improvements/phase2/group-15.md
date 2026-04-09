# Cross-Cutting Patterns (Group 15)

**Slices analyzed**: apps-app-src-hooks-1, apps-app-src-hooks-2, apps-app-src-hooks-3, apps-app-src-hooks-4, apps-app-src-hooks-5, apps-app-src-hooks-6, apps-app-src-hooks-7, apps-app-src-hooks-8, apps-app-src-hooks-9, apps-app-src-hooks-10, apps-app-src-hooks-11, apps-app-src-lib-1

## Pattern 1: Error body casts instead of `extractErrorMessage`

- **Seen in**: hooks-3 (useBehavioralFingerprint, useBulkActions), hooks-6 (useKnowledgeSourceArchive x3, useKnowledgeSources x5), hooks-10 (useTriageTrigger, useUpdateIntegration), lib-1 (ai-chat-controller, api-client)
- **Category**: type-cast
- **Combined impact**: high -- at least 13 instances across 8+ files
- **What's happening**: Non-200 response bodies are cast to `{ message?: string }`, `{ message?: string | string[] }`, `{ error?: string }`, or `Record<string, unknown>` to extract error messages. The field names are inconsistent (`message` vs `error`), and the cast shapes vary (some include array variants, some don't). Meanwhile, `extractErrorMessage` from `@slopweaver/contracts` already handles all these variants safely.
- **Suggestion**: Replace every `response.body as { message?: ... }` pattern with `extractErrorMessage` from `@slopweaver/contracts`. This eliminates all error body casts and standardizes on a single extraction path. A codemod searching for `as { message` and `as { error` would catch most instances.
- **Estimated scope**: ~13 instances across 8 files in hooks/ and lib/

## Pattern 2: `platform: string` instead of `PlatformId` in local interfaces

- **Seen in**: hooks-1 (entities-data.utils.ts), hooks-2 (proposals.utils.ts, queue-data-transformers.ts), hooks-4 (useExpandedContent.ts)
- **Category**: sdk-type-duplication
- **Combined impact**: medium -- 4+ local interfaces use `string` where `PlatformId` union exists
- **What's happening**: Local interfaces that describe API response shapes use `platform: string` instead of importing `PlatformId` from `@slopweaver/contracts`. This permits invalid platform values at compile time and defeats exhaustive switch/match patterns downstream.
- **Suggestion**: Import `PlatformId` from `@slopweaver/contracts` and replace `platform: string` in all local interfaces. When the local interface is itself a duplicate of a contract type (see Pattern 3), this fix comes for free by deleting the local type.
- **Estimated scope**: 4-6 interfaces across 4 files

## Pattern 3: Local interfaces duplicating contract response types

- **Seen in**: hooks-1 (entities-data.utils.ts: ApiEntityIdentity, ApiEntity, ApiPendingLink), hooks-2 (proposals.utils.ts: ApiProposal, queue-data-transformers.ts: QueueWorkItemTransformInput), hooks-3 (useAnalyticsData.ts: 4 stats response interfaces), hooks-7 (usePatternLearningProgress.ts: PatternLearningProgress), hooks-8 (usePriorityScoring.ts: VipContact, useProactivePreferences.ts: ProactivePreferences), hooks-9 (useSettingsData.ts: UserSettings, UpdateSettingsInput), hooks-10 (useTodosData.ts: Task, TaskCategory, TaskEffort, TaskPriority, TaskStatus), lib-1 (ai-chat-types.ts: ToolCall)
- **Category**: sdk-type-duplication
- **Combined impact**: high -- 20+ local type definitions across 10+ files
- **What's happening**: Hooks define their own interfaces for API response shapes instead of importing from `@slopweaver/contracts`. These local types can silently drift from the actual API schema. Some are exact duplicates; others are intentional adaptations (e.g., Task vs TodoResponse with different priority values) that lack documentation explaining the divergence.
- **Suggestion**: Audit each local type against `@slopweaver/contracts`. For exact duplicates, delete and import. For intentional adaptations (like Task), document the mapping rationale and consider using `Pick`/`Omit`/intersection types derived from the contract type. This is the single largest type-safety improvement opportunity in the hooks layer.
- **Estimated scope**: 20+ type definitions across 10+ files

## Pattern 4: `BackendIntegration` defined in 3 separate files

- **Seen in**: hooks-5 (useIntegrationsDirectory.ts), hooks-5 (useIntegrationHub.ts), hooks-7 (usePlatformSync.ts)
- **Category**: duplicate-type
- **Combined impact**: high -- 3 identical definitions of the same interface
- **What's happening**: `BackendIntegration` is independently defined in three hook files. All three describe the same API response shape. When the backend changes, all three must be updated manually, and any divergence is silent.
- **Suggestion**: Extract to a single canonical location -- either `@slopweaver/contracts` (preferred) or `apps/app/src/types/integration.ts`. Delete all three local copies.
- **Estimated scope**: 3 files, 1 shared type definition to create

## Pattern 5: `Record<string, number>` for platform/status count maps

- **Seen in**: hooks-2 (queue-data-transformers.ts: buildPlatformCounts, buildStatusCounts), hooks-8 (useQueueData.ts: platformCounts, statusCounts), hooks-9 (useSearchData.ts: platformCounts)
- **Category**: record-weakening
- **Combined impact**: medium -- 5+ instances across 3 files
- **What's happening**: Count maps keyed by platform ID or work item status use `Record<string, number>` instead of `Partial<Record<PlatformId, number>>` or `Partial<Record<WorkItemStatus, number>>`. This allows accessing counts with arbitrary string keys without TypeScript flagging the mistake.
- **Suggestion**: Replace `Record<string, number>` with `Partial<Record<PlatformId, number>>` for platform counts and `Partial<Record<WorkItemStatus, number>>` for status counts. The `Partial` wrapper correctly models that not every key will be present.
- **Estimated scope**: 5 instances across 3 files

## Pattern 6: Mutation `mutateAsync` returning `Promise<unknown>` instead of typed response

- **Seen in**: hooks-4 (useEntitiesData.ts: 5 mutations), hooks-6 (useMemoryFiles.ts: 3 mutations), hooks-8 (useQueueData.ts: 2 mutations), hooks-9 (useSettingsData.ts: 1 mutation), hooks-10 (useTodoMutations.ts: 3 mutations, useTodoUpdateMutation.ts: 1 mutation)
- **Category**: any-usage
- **Combined impact**: high -- 15+ mutation return types lose their contract-derived type
- **What's happening**: Custom hook return interfaces declare `mutateAsync` as returning `Promise<unknown>`, even though the underlying `mutationFn` returns a typed value from the ts-rest contract. Callers who need to inspect the result (e.g., to get a created entity's ID) must cast or ignore the return value.
- **Suggestion**: Derive explicit return types from the contract response schemas. For each mutation, the return type should be `Promise<ContractResponseBodyType>` (where the body type is inferred from `tsr.<endpoint>.mutate`). This is a mechanical change: for each `Promise<unknown>`, look at what `mutationFn` actually returns and propagate that type.
- **Estimated scope**: 15+ mutation interfaces across 6 files

## Pattern 7: `as SomeType` casts on ts-rest response bodies

- **Seen in**: hooks-1 (useBillingCommerce.ts), hooks-4 (useExpandedContent.ts), hooks-6 (useKnowledgeGraphData.ts, useKnowledgeSources.ts), hooks-8 (usePriorityScoring.ts x2), hooks-9 (useSyncStream.ts)
- **Category**: type-cast
- **Combined impact**: medium -- 8+ instances across 6 files
- **What's happening**: After a successful ts-rest API call, `response.body` is cast to the expected type (e.g., `response.body as VipContact`, `body.edges as KnowledgeGraphEdge[]`). This suggests the ts-rest contract's 200 response body is not propagating the correct type to the client. Either the contract is typed too loosely, or the hook is not using `select: (d) => d.body` to let TypeScript infer the body type.
- **Suggestion**: Ensure each ts-rest contract endpoint defines its 200 response body schema precisely. Then use `select: (d) => d.body` on `useQuery` calls or check `response.status === 200` to let TypeScript narrow the body type automatically. Files like `useTriageStats.ts` and `useSuggestedActions.ts` already do this correctly and serve as examples.
- **Estimated scope**: 8+ instances across 6 files

## Pattern 8: `unknown` parameter types pushing narrowing burden to consumers

- **Seen in**: hooks-3 (useChatPersistence.ts: `user?: unknown`), hooks-4 (useChatTransport.ts: `currentContext?: unknown`), lib-1 (app-updates/orchestrator.ts: `cachedTauriUpdate: unknown`)
- **Category**: any-usage
- **Combined impact**: medium -- 3 instances, each requiring downstream casts
- **What's happening**: Function parameters or module-level variables are typed as `unknown` when their actual shape is well-known. Downstream code must narrow via type guards or casts, duplicating knowledge of the shape at every access site.
- **Suggestion**: Type these with their actual shapes: `user` as `{ id: string } | null`, `currentContext` as `CurrentContext` from ai-chat-types, `cachedTauriUpdate` as `import type { Update } from '@tauri-apps/plugin-updater' | null`. Each eliminates multiple downstream narrowing operations.
- **Estimated scope**: 3 variables/parameters across 3 files

## Pattern 9: AI SDK `UIMessage` extended via inline intersection casts

- **Seen in**: hooks-3 (useChatPersistence.ts: `as UIMessage`), hooks-4 (useChatToolsTimeline.ts: `msg.parts as MessagePart[]`, `msg as UIMessage & { serverTimestamp?: string }`, `msg as UIMessage & { isVoiceTranscript?: boolean }`)
- **Category**: type-cast
- **Combined impact**: medium -- 4+ casts across 2 files, likely more in unaudited hooks
- **What's happening**: The Vercel AI SDK's `UIMessage` type does not include SlopWeaver-specific fields (`serverTimestamp`, `isVoiceTranscript`) or have precisely typed `parts`. Multiple hooks independently cast `UIMessage` to access these extensions, creating fragile inline type augmentations.
- **Suggestion**: Create a central `SlopweaverMessage` type (or module augmentation of `UIMessage`) in `ai-chat-types.ts` that adds `serverTimestamp?: string`, `isVoiceTranscript?: boolean`, and correctly typed `parts`. All hooks import this augmented type instead of casting `UIMessage` inline.
- **Estimated scope**: 4+ inline casts across 2+ files; centralized fix in 1 file

## Pattern 10: `Record<string, unknown>` for structured metadata/settings

- **Seen in**: hooks-5 (useInboxData.ts: InboxMessageMetadata), hooks-7 (usePatternLearningStream.ts: event.metadata), hooks-10 (useUpdateIntegration.ts: syncSettings), lib-1 (analytics-events.ts: AnalyticsPayload)
- **Category**: record-weakening
- **Combined impact**: medium -- 4 instances across 4 files
- **What's happening**: Structured objects with known fields are typed as `Record<string, unknown>`, forcing bracket-access that returns `unknown`. In each case, the code immediately accesses specific known keys, proving the shape is not truly open-ended.
- **Suggestion**: For each instance, define the known fields explicitly on the type. Use a discriminated union (for metadata with platform-specific shapes) or explicit optional fields (for settings). Keep a `& Record<string, unknown>` intersection only if genuinely open-ended extension is needed.
- **Estimated scope**: 4 types across 4 files
