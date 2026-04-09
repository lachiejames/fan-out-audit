# Cross-Cutting Patterns (Group 5)

**Slices analyzed**: apps-api-src-domain-5, apps-api-src-domain-6, apps-api-src-domain-7, apps-api-src-domain-8, apps-api-src-infrastructure-1, apps-api-src-infrastructure-2, apps-api-src-infrastructure-3, apps-api-src-infrastructure-4, apps-api-src-infrastructure-5, apps-api-src-infrastructure-6, apps-api-src-infrastructure-7, apps-api-src-infrastructure-8

## Pattern 1: Copy-pasted `result as { usage?: ... }` token-usage extraction casts

- **Seen in**: infrastructure-1 (Findings 1-3), infrastructure-4 (Findings 1, 6), infrastructure-5 (Finding 1)
- **Category**: type-cast
- **Combined impact**: high -- appears in 8+ locations across 6 files, each silently discarding well-typed SDK return values
- **What's happening**: Every AI adapter and document-intelligence service that calls `generateText()` (Vercel AI SDK) or Voyage/Anthropic APIs extracts token usage by casting the already-typed result to an inline structural type like `{ usage?: { input_tokens?: number; output_tokens?: number } }` or `{ usage?: { totalTokens?: number } }`. The SDK responses already carry typed `usage` fields (`LanguageModelUsage`, `EmbeddingsResponse`, etc.), so these casts downgrade compile-time safety to runtime-only checks. The pattern was clearly copy-pasted -- the code is character-for-character identical across all sites.
- **Suggestion**: Extract two shared helpers: (1) `extractVercelAiTokenUsage(result: GenerateTextResult<any, any>)` for the Vercel AI SDK pattern, and (2) `extractVoyageTokenUsage(result: EmbeddingsResponse | ContextualizedEmbeddingsResponse | MultimodalEmbeddingsResponse)` for the Voyage pattern. Each adapter then calls the typed helper instead of inlining a cast. This eliminates ~8 identical cast blocks and surfaces SDK breaking changes at compile time.
- **Estimated scope**: 8 instances across 6 files (3 AI adapters, 4 document-intelligence services, 1 embedding adapter)

## Pattern 2: Duplicate `subscriptionStatus` literal union inline across domain and infrastructure

- **Seen in**: domain-5 (Finding 1), infrastructure-1 (Finding 5)
- **Category**: duplicate-type
- **Combined impact**: high -- the six-value union `"trialing" | "active" | "past_due" | "cancelled" | "expired" | "paused"` appears 11+ times across the codebase
- **What's happening**: The `SubscriptionStatus` type already exists in `@slopweaver/contracts` (derived from `subscriptionStatusSchema`), yet the identical literal union is inlined in `ai-model.port.ts` (2x), `ai-work-item-generation.port.ts` (1x), and `ai-model.utils.ts` (4x), plus likely other files. Adding or renaming a status value requires touching all 11+ locations independently.
- **Suggestion**: Import `SubscriptionStatus` from `@slopweaver/contracts` everywhere. The domain layer already imports from contracts in other ports (e.g. `billing.port.ts`), so there is no layer violation. A single find-and-replace of the inline union with the imported type eliminates all 11 instances.
- **Estimated scope**: 11+ inline occurrences across 4+ files

## Pattern 3: Positional parameters in domain port abstract methods (violates named-object-params rule)

- **Seen in**: domain-6 (Findings 1, 2, 5), domain-7 (Findings 3, 4, 5)
- **Category**: missing-strict-typing
- **Combined impact**: medium -- at least 10 abstract methods across 6 port files use positional params where the codebase mandates named-object params
- **What's happening**: Multiple domain ports define abstract methods with bare positional parameters (e.g. `getContext(userId: string)`, `read(userId: string, path: string)`, `getSettings(userId: string)`, `canProceedWithAICall(userId: string)`). Several have two same-typed string params that can be silently swapped. Other ports in the same directory correctly use named-object params, creating inconsistency.
- **Suggestion**: Systematically refactor all positional-param abstract methods in `apps/api/src/domain/ports/services/` to named-object params. This is a mechanical change: wrap each parameter list in `{ paramName }: { paramName: Type }`. Update all implementing classes and call sites. Can be done one port file at a time.
- **Estimated scope**: ~10 methods across 6 port files (llm-context, memory, settings, profile-context, token-cap, integrations-lookup), plus their implementations and call sites

## Pattern 4: `Record<string, unknown>` used where domain types or contract types would be safer

- **Seen in**: domain-5 (Finding 3), domain-7 (Finding 6), infrastructure-2 (Findings 4, 5), infrastructure-3 (Findings 3, 5), infrastructure-6 (Finding 1)
- **Category**: record-weakening
- **Combined impact**: medium -- at least 7 instances where `Record<string, unknown>` erases known structure
- **What's happening**: Multiple locations use `Record<string, unknown>` for values that have a known shape at runtime: `getMonthlyUsageByPlatform` returns `Record<string, number>` where keys are platform IDs; `serializeProfileForCache` accepts `Record<string, unknown>` when callers always pass `CoreProfile`; `buildPostHogPersonProperties` returns `Record<string, unknown>` with a predictable set of keys; `logData` in HTTP middleware has fixed known keys; `statusCounts`/`healthCounts` accumulators use `Record<string, number>` where keys are typed enum values. In each case, a narrower type (contract enum as key, named interface for known keys) would catch bugs at compile time.
- **Suggestion**: Address in priority order: (1) Replace `Record<string, number>` with `Partial<Record<IntegrationPlatform, number>>` and `Partial<Record<IntegrationStatus, number>>` where contract types exist. (2) Add named interfaces for `PostHogPersonProperties` and `HttpLogPayload` with known keys intersected with `Record<string, unknown>` for extensibility. (3) Tighten `serializeProfileForCache` to accept `CoreProfile` or `Record<string, JsonValue>`.
- **Estimated scope**: 7+ instances across 7 files in domain and infrastructure layers

## Pattern 5: `as SomeType` casts on `.includes()` calls for readonly typed arrays

- **Seen in**: infrastructure-1 (Finding 4), infrastructure-2 (Findings 1, 3)
- **Category**: type-cast
- **Combined impact**: medium -- identical boilerplate in 6+ type-guard functions
- **What's happening**: Every `isSupportedXType` function (audio, pptx, rtf, spreadsheet, vision, zip) uses the same pattern: `SUPPORTED_X_TYPES.includes(mimeType as XMediaType)`. The cast exists because TypeScript's `ReadonlyArray<T>.includes()` only accepts `T`, not the wider `string`. Each file independently defines its own constant, type, and guard function with the same structure.
- **Suggestion**: Create a single generic helper: `function isMember<T extends string>(arr: readonly T[], value: string): value is T { return (arr as readonly string[]).includes(value); }`. All 6+ type-guard functions can then call `isMember(SUPPORTED_X_TYPES, mimeType)` with zero casts at the call site. Alternatively, create a `createMimeTypeGuard<T extends string>(supported: readonly T[])` factory that returns the type-predicate function.
- **Estimated scope**: 6 files in `apps/api/src/infrastructure/ai/utils/` (audio, pptx, rtf, spreadsheet, vision, zip content utils)

## Pattern 6: Redundant `as DomainType` casts after Zod validation

- **Seen in**: infrastructure-4 (Findings 2, 3, 4)
- **Category**: type-cast
- **Combined impact**: medium -- 3 instances in document-intelligence services, likely more elsewhere
- **What's happening**: After successful Zod `.safeParse()` or `.parse()`, the validated `data` is cast back to a domain type with `as ContractAnalysis`, `as ExtractedEntities`, or `as DocumentSummary`. If the domain type is structurally identical to `z.infer<typeof schema>`, the cast is unnecessary. If they differ, the cast silently hides the mismatch -- which is worse.
- **Suggestion**: Derive the domain types directly from the Zod schemas: `type ContractAnalysis = z.infer<typeof contractAnalysisSchema>`. This eliminates the cast and guarantees structural equivalence by construction. Alternatively, use `satisfies DomainType` instead of `as DomainType` to get a compile error on mismatch rather than silent suppression.
- **Estimated scope**: 3 confirmed instances in document-intelligence; likely more across the codebase wherever Zod schemas are used alongside separate domain type definitions

## Pattern 7: `any[]` usage in NestJS queue/module factory patterns

- **Seen in**: infrastructure-7 (Finding 4), infrastructure-8 (Findings 1, 2)
- **Category**: any-usage
- **Combined impact**: medium -- 3 instances in queue infrastructure, affects all queue modules built by the factories
- **What's happening**: `createQueueService` returns `new (...args: any[]) => QueueServiceInstance<T>` and `createQueueModule` uses `any[]` for both its `queueService` parameter union and its `exports` array. These `any[]` values propagate through every queue module in the system. While partly a NestJS DI limitation (constructor args are resolved at runtime), the `any[]` suppresses compile-time checking of the module factory's inputs.
- **Suggestion**: (1) Narrow the constructor signature to `new (queue: Queue<T>) => QueueServiceInstance<T>` in `createQueueService`. (2) Remove the second union branch in `createQueueModule`'s `queueService` parameter and require `Type<QueueServiceInstance<TJob>>`. (3) Replace `const exports: any[]` with `const exports: Array<Type | DynamicModule>` using NestJS types.
- **Estimated scope**: 2 factory files, but the fix propagates type safety to all ~10 queue service files that use the factories

## Pattern 8: Untyped queue services defaulting to `Queue<unknown>`

- **Seen in**: infrastructure-7 (Findings 1, 2, 3), infrastructure-8 (Finding 3)
- **Category**: missing-strict-typing
- **Combined impact**: high -- at least 2 queue services accept arbitrary payloads at compile time, and the webhook queue is the most critical (handles all platform webhooks)
- **What's happening**: `WebhooksQueueService` and `PatternExtractionQueueService` call `createQueueService()` without a type parameter, defaulting to `T = unknown`. This means `queue.add(name, data)` accepts any value for `data` with no compile-time verification. The webhook queue is particularly dangerous because it processes payloads from Slack, Gmail, Linear, etc. -- a structurally wrong payload would be silently enqueued and only fail at processing time.
- **Suggestion**: Define discriminated union job types for each queue (e.g. `type WebhookJob = SlackWebhookJob | GmailWebhookJob | LinearWebhookJob`) in the respective constants files, and pass them as the generic parameter: `createQueueService<WebhookJob>({ queueName: WEBHOOKS_QUEUE })`. The `enqueueWebhookJob` method in `BackgroundJobsAdapter` should also be updated from `data: unknown` to `data: WebhookJob`.
- **Estimated scope**: 2 queue service files + 1 adapter method + constants files for job type definitions

## Pattern 9: Duplicate type definitions where contracts package already exports the canonical type

- **Seen in**: domain-7 (Findings 1, 2), domain-8 (Finding 2), infrastructure-4 (Finding 5)
- **Category**: duplicate-type
- **Combined impact**: medium -- at least 6 inline type definitions duplicate types from `@slopweaver/contracts`
- **What's happening**: Beyond the subscription-status union (Pattern 2), several other types are duplicated: `todos.port.ts` re-declares `TodoStatus`, `TodoPriority`, `TodoEffort`, and `TodoCategory` inline instead of importing from contracts; `TodoResult` widens these to plain `string`; `ocr.service.ts` re-declares `VisionMediaType` that is already exported from `vision-content.utils.ts`; `getRegisteredPlatforms()` returns `string[]` instead of `PlatformId[]`. The common theme is that narrower, canonical types exist but are not being used at the boundary.
- **Suggestion**: Audit all domain port files and infrastructure services for types that shadow or widen contracts-package types. Replace inline unions with imports from `@slopweaver/contracts`. Replace `string` return types with their narrower contract equivalents (`PlatformId`, `TodoStatus`, etc.).
- **Estimated scope**: 6+ type definitions across 4 files (todos.port.ts, ocr.service.ts, plugin-registry.port.ts, ai-model.port.ts)

## Pattern 10: `unknown` used for cross-layer boundary types (domain ports accepting infrastructure objects)

- **Seen in**: domain-5 (Finding 4), domain-6 (Finding 3)
- **Category**: any-usage
- **Combined impact**: low -- architecturally motivated but could be narrowed
- **What's happening**: Domain ports use `unknown` for parameters that represent infrastructure-layer types to avoid importing NestJS/Express types into the domain layer. Examples: `CreateClaudeModelParams.logger?: unknown` (should be the domain's own `ILoggerPort`), `OAuthHandler.query: unknown` and `request/response: unknown` (NestJS/Express objects). While the architectural reasoning is sound, `unknown` accepts literally any value including primitives and `null`, which could mask bugs.
- **Suggestion**: For `logger`, use the domain's own `ILoggerPort` interface. For `query`, use `Record<string, unknown>` to at least enforce it is an object. For `request`/`response`, `unknown` is acceptable since these truly are opaque infrastructure objects, but add JSDoc comments documenting the expected types (`@param request Express.Request`).
- **Estimated scope**: 4 parameters across 2 port files
