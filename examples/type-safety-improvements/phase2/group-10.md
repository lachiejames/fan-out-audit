# Cross-Cutting Patterns (Group 10)

**Slices analyzed**: apps-api-src-interface-21, apps-api-src-interface-22, apps-api-src-interface-23, apps-api-src-interface-3, apps-api-src-interface-4, apps-api-src-interface-5, apps-api-src-interface-6, apps-api-src-interface-7, apps-api-src-interface-8, apps-api-src-interface-9, apps-api-src-seed-data-1, apps-api-src-seed-data-2

## Pattern 1: `Record<string, unknown>` used as a catch-all for structured data with known shapes

- **Seen in**: apps-api-src-interface-3 (ToolReviewEditMapper toolInput), apps-api-src-interface-4 (edit-knowledge.tool.ts data object, context-formatting.utils.ts EnrichedContext.metadata), apps-api-src-interface-5 (workspace-agent.service.ts tools map), apps-api-src-interface-7 (chat-pipeline.ts tools field), apps-api-src-interface-8 (chat-stream-callbacks.utils.ts OnFinishContext.tools), apps-api-src-interface-21 (webhook.processor.ts platformMetadata), apps-api-src-interface-23 (webhook-job.utils.ts WebhookJob union), apps-api-src-seed-data-1 (sync-checkpoints.ts CursorState, knowledge-items.ts embeddings map)
- **Category**: record-weakening
- **Combined impact**: High -- this is the single most pervasive type-safety gap across these slices. At least 9 distinct instances across 8 slices use `Record<string, unknown>` where the actual shape is known at compile time. This defeats TypeScript's structural checking: typos in keys, wrong value types, and missing required fields all pass silently.
- **What's happening**: Multiple subsystems use `Record<string, unknown>` as a convenient "any object" type for data that actually has a well-defined structure. Examples include: tool input maps where Zod schemas already define the shape, platform metadata where per-platform interfaces are known, cursor state objects where each platform has a distinct shape, and AI SDK tool dictionaries where `ToolSet` from the `ai` package exists.
- **Suggestion**: Replace each instance with the narrowest available type: (1) For tool inputs, make `ToolReviewEditMapper` generic over the Zod schema's inferred type. (2) For platform metadata, define per-platform interfaces (`GmailPlatformMetadata`, `SlackPlatformMetadata`, etc.) and narrow via discriminant. (3) For cursor states, define per-platform cursor interfaces. (4) For AI SDK tool maps, use `ToolSet` from the `ai` package. (5) For `EnrichedContext.metadata`, define a typed interface with known fields plus an index signature.
- **Estimated scope**: ~15-20 instances across ~12 files

## Pattern 2: Type casts after Zod validation instead of using `parse.data` directly

- **Seen in**: apps-api-src-interface-6 (auth.controller.ts `platform as "web" | "tauri-desktop" | "tauri-mobile"` after safeParse), apps-api-src-interface-7 (calendar-response.utils.ts `platform as CalendarEventItem["platform"]`, chat-citations.utils.ts `as CitationSourceType` after safeParse), apps-api-src-interface-9 (chat.controller.ts `item.rating as "positive" | "negative"`)
- **Category**: type-cast
- **Combined impact**: Medium -- these casts are structurally safe (the value was validated), but they bypass TypeScript's narrowing and create maintenance risk. If the Zod schema changes, the cast silently accepts the old union, hiding the mismatch until runtime.
- **What's happening**: After calling `.safeParse()` or `.parse()`, the code casts the original untyped variable to the expected literal union instead of using the already-narrowed `parsed.data`. This pattern appears at authentication boundaries, calendar response mapping, and citation source classification.
- **Suggestion**: Replace `originalVar as UnionType` with `parsed.data` from the Zod parse result. Where the value comes from a DB column typed as `string` but constrained to a known set (e.g., `rating`), either use a `pgEnum` Drizzle column type or add a Zod parse at the DB boundary.
- **Estimated scope**: ~6-8 instances across ~4 files

## Pattern 3: `LoggerLike` interface duplicated 4 times across the chat module

- **Seen in**: apps-api-src-interface-7 (chat-pipeline.ts), apps-api-src-interface-8 (chat-stream-callbacks.utils.ts, chat-stream-guards.utils.ts, chat-stream.handler.ts)
- **Category**: duplicate-type
- **Combined impact**: High -- four identical copies of the same interface in the same module. Any change to the logger contract requires updating all four files, and drift between them would cause silent type incompatibilities.
- **What's happening**: Each file in the chat module independently defines `LoggerLike` with the same shape (`info`, `warn`, `error`, `debug` methods). This appears to be copy-paste to avoid circular imports or because no shared types file exists for the chat module.
- **Suggestion**: Extract `LoggerLike` to a single shared location such as `apps/api/src/interface/http/chat/chat.types.ts` or `apps/api/src/shared/types/logger-like.ts` and import from all four files.
- **Estimated scope**: 4 files, 1 interface definition each

## Pattern 4: Inline structural casts (`as { field?: unknown }`) to access properties on `unknown` or loosely-typed objects

- **Seen in**: apps-api-src-interface-3 (get-notion-page.tool.ts multiple property casts), apps-api-src-interface-4 (edit-knowledge.tool.ts, update-notion-page.tool.ts formatPropertyValue), apps-api-src-interface-5 (workspace-agent.service.ts token usage access), apps-api-src-interface-8 (chat-stream-callbacks.utils.ts ToolResult args access, chat-stream.utils.ts getObjectRecord), apps-api-src-interface-21 (webhook.processor.ts organizationId extraction), apps-api-src-interface-23 (webhooks-http.helpers.ts signedPayload access)
- **Category**: type-cast
- **Combined impact**: High -- this is the most common cast pattern, appearing in at least 7 slices. Each instance creates an inline anonymous type just to access one or two properties on an `unknown` value. The casts skip runtime validation, so a missing or wrong-typed field silently returns `undefined`.
- **What's happening**: When a value is typed as `unknown` (from DB JSON columns, BullMQ job data, AI SDK results, or webhook payloads), the code uses `(value as { field?: Type }).field` to access properties. The structural check that often precedes the cast (`typeof === "object"`, `"field" in obj`) is not leveraged by TypeScript's control flow analysis because the full narrowing is not completed.
- **Suggestion**: Two approaches: (1) Define reusable type guard functions (e.g., `hasField<K>(obj: unknown, key: K): obj is Record<K, unknown>`) that complete the narrowing without casts. (2) For structured data from known sources (AI SDK, Notion SDK, BullMQ), import the SDK's exported types and use them directly rather than casting to ad-hoc inline shapes.
- **Estimated scope**: ~15-20 cast sites across ~10 files

## Pattern 5: BullMQ `Job<UnionType>` forces per-case type casts in switch statements

- **Seen in**: apps-api-src-interface-21 (webhook.processor.ts -- 10 casts), apps-api-src-interface-23 (webhook-job.utils.ts -- duplicated job types)
- **Category**: type-cast + duplicate-type
- **Combined impact**: Medium -- BullMQ's `Job<T>` wraps data rather than spreading it, so `switch (job.name)` cannot narrow the generic parameter. This forces 10 explicit `as Job<SpecificType>` casts in the webhook processor. The job data type definitions are then duplicated in a utils file because they were not extracted to a shared location.
- **What's happening**: The webhook processor handles 10+ job types via a switch on `job.name`. Each case branch requires a manual cast because TypeScript cannot discriminate `Job<A | B | C>` through `job.name`. A separate utils file copies the same type definitions rather than importing them, and adds `| Record<string, unknown>` which widens the entire union.
- **Suggestion**: (1) Extract all webhook job data interfaces to a shared `webhook-job-data.ts` types file. (2) Create a centralized type-narrowing module with per-job-type assertion functions. (3) Remove the `| Record<string, unknown>` escape hatch from the union in `webhook-job.utils.ts`.
- **Estimated scope**: ~12 cast sites across 2 files, ~4 duplicated interfaces

## Pattern 6: `as UIMessage["parts"]` cast used in multiple chat utility files

- **Seen in**: apps-api-src-interface-8 (chat-stream.utils.ts lines 114, 300), apps-api-src-interface-9 (chat-ui-message-preprocess.utils.ts lines ~80, ~300)
- **Category**: type-cast
- **Combined impact**: Medium -- the Vercel AI SDK's `UIMessage["parts"]` is a complex array union type. After building parts arrays, TypeScript cannot infer they satisfy the union, so both files use `as UIMessage["parts"]`. This is structurally sound but silently accepts wrong shapes.
- **What's happening**: Two separate files build UI message parts arrays and cast them to the SDK's expected type. The cast hides any structural mismatch between what the code builds and what the SDK expects.
- **Suggestion**: Replace `as UIMessage["parts"]` with `satisfies UIMessage["parts"]` for compile-time structural checking. Alternatively, extract a shared `buildUiMessageParts()` helper that returns the correctly-typed array, eliminating the cast in both call sites.
- **Estimated scope**: 4 cast sites across 2 files

## Pattern 7: Express `Request` header/cookie access requires repeated casts due to loose SDK types

- **Seen in**: apps-api-src-interface-5 (concurrency-cleanup.interceptor.ts `request.concurrencyKey as string`), apps-api-src-interface-6 (auth.controller.ts -- 5+ header casts, cookie casts, app.get() cast)
- **Category**: type-cast
- **Combined impact**: Low -- Express types all headers as `string | string[] | undefined` and cookies as `any`. Every header access requires a cast. These casts are unavoidable given Express's type definitions, but the repetition creates noise and inconsistency.
- **What's happening**: Express's `@types/express` definitions are intentionally loose for headers and cookies. Each access to `req.headers["x-platform"]`, `req.cookies`, or custom request properties like `concurrencyKey` requires a cast.
- **Suggestion**: (1) Create a `getStringHeader({ req, name })` helper that encapsulates the cast once. (2) Extend the Express `Request` interface with an ambient declaration for custom properties like `concurrencyKey`. (3) For `req.cookies`, create a typed cookie accessor.
- **Estimated scope**: ~8-10 cast sites across 2-3 files
