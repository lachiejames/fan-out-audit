# Cross-Cutting Patterns (Group 6)

**Slices analyzed**: apps-api-src-infrastructure-9, apps-api-src-integrations-1, apps-api-src-integrations-10, apps-api-src-integrations-11, apps-api-src-integrations-12, apps-api-src-integrations-13, apps-api-src-integrations-14, apps-api-src-integrations-15, apps-api-src-integrations-16, apps-api-src-integrations-17, apps-api-src-integrations-18, apps-api-src-integrations-19

## Pattern 1: BaseSyncService forces `unknown[]` on override methods, requiring casts in every platform

- **Seen in**: integrations-11 (Asana), integrations-14 (GitHub), integrations-16 (Google Docs), integrations-17 (Google Drive), integrations-18 (Google Gmail), integrations-19 (Google Gmail hooks)
- **Category**: type-cast
- **Combined impact**: high -- affects every sync service in the codebase (6+ occurrences found, likely more in unaudited platforms). Each subclass immediately casts `parsedItems as ResolvedAsanaTask[]`, `ParsedGoogleDoc[]`, `ParsedGmailMessage[]`, etc. The casts are structurally identical and all stem from the same root cause.
- **What's happening**: `BaseSyncService` declares `getEmbeddingTexts(parsedItems: unknown[]): string[]`, `syncAttachments(ctx, insertedIds, parsedItems: unknown[])`, and `onContentInserted(ctx, insertedIds, parsedItems: unknown[])` with `unknown[]` parameters. Every platform override must immediately cast to its concrete parsed-item type.
- **Suggestion**: Add a generic type parameter to `BaseSyncService<..., ParsedItem>` so override signatures become `getEmbeddingTexts(parsedItems: ParsedItem[]): string[]`. This eliminates all downstream casts in one change.
- **Estimated scope**: 1 base class change + ~10-15 sync service files across all platforms

## Pattern 2: `integration.platformMetadata` typed `unknown` in DB schema forces repeated casts in every OAuth/plugin/sync service

- **Seen in**: integrations-10 (Asana), integrations-11 (Asana OAuth), integrations-12 (Facebook Messenger plugin, OAuth), integrations-15 (Google Calendar OAuth), integrations-16 (Google Docs OAuth, Google Drive OAuth), integrations-18 (Google Gmail OAuth, sync config)
- **Category**: type-cast
- **Combined impact**: high -- the `platformMetadata` column is `unknown` (jsonb), so every service that reads it must cast: `integration.platformMetadata as Partial<AsanaPlatformMetadata>`, `as GoogleCalendarPlatformMetadata | null`, `as { pageId?: string } | null`, etc. At least 8 distinct cast sites found in the audited slices alone; likely 15-20+ across all platforms.
- **What's happening**: The database schema types `platformMetadata` as `unknown` (correct for a jsonb column), but there is no centralized parse-and-validate step. Each service invents its own inline cast with slightly different shapes (`Partial<T>`, `T | null`, `{ field?: string } | null`).
- **Suggestion**: For each platform, define a Zod schema that matches the platform metadata interface (e.g., `asanaPlatformMetadataSchema`). Add a shared utility `parsePlatformMetadata<T>(raw: unknown, schema: ZodType<T>): T | null` and call it once at the service entry point. This replaces all inline casts with validated parsing.
- **Estimated scope**: ~10-12 platform metadata schemas to create + ~20 cast sites to replace with parse calls

## Pattern 3: `PlatformMetadata extends Record<string, unknown>` weakens typed interfaces

- **Seen in**: integrations-10 (AsanaPlatformMetadata), integrations-16 (GoogleDocsPlatformMetadata, GoogleDrivePlatformMetadata)
- **Category**: record-weakening
- **Combined impact**: medium -- at least 3 platform metadata interfaces use `extends Record<string, unknown>` (some with redundant `[key: string]: unknown`), which adds an index signature that allows any string key to compile without error. This undermines the purpose of having typed fields.
- **What's happening**: The `extends Record<string, unknown>` pattern was likely added so the interface is assignable to the `unknown` jsonb column type. But it weakens all the typed fields by allowing arbitrary key access.
- **Suggestion**: Remove the `Record<string, unknown>` extension from all platform metadata interfaces. Instead, cast to `unknown` (or `Json`) only at the database write boundary: `metadata as unknown as Json`. This preserves full type safety everywhere except the single DB insertion point.
- **Estimated scope**: ~5-8 platform metadata interfaces across the codebase

## Pattern 4: `filter(Boolean) as T[]` instead of type-predicate filter

- **Seen in**: integrations-11 (Asana sync-fetch), integrations-15 (Google Calendar sync), integrations-17 (Google Drive sync)
- **Category**: type-cast
- **Combined impact**: low -- 3 occurrences found, likely a few more in unaudited slices. Each uses `.filter(Boolean) as ConcreteType[]` because TypeScript cannot narrow element types through `Boolean` as a filter predicate.
- **What's happening**: After `.map()` that may return `undefined`, `.filter(Boolean)` removes nullish values but TypeScript's type system does not narrow the array element type. A cast is required to get back to the concrete type.
- **Suggestion**: Replace `.filter(Boolean) as T[]` with `.filter((x): x is T => x != null)`. This is a mechanical find-and-replace that eliminates the cast entirely.
- **Estimated scope**: ~5-8 instances across sync services

## Pattern 5: `fetchThread` returns `Promise<unknown>` across sync services

- **Seen in**: integrations-16 (Google Docs sync), integrations-17 (Google Drive sync), integrations-18 (Google Gmail sync)
- **Category**: missing-strict-typing
- **Combined impact**: medium -- at least 3 sync services return `Promise<unknown>` from `fetchThread`, losing the concrete thread/document shape. Callers must cast or treat the result as opaque. This is likely a base class signature issue similar to Pattern 1.
- **What's happening**: `fetchThread()` constructs a well-defined return object (with fields like id, name, content, mimeType, etc.) but declares the return type as `Promise<unknown>` or `Promise<Result<unknown, IntegrationError>>`. The actual shapes differ per platform but are always known at the implementation site.
- **Suggestion**: If `fetchThread` is a base class method, add a generic type parameter for the return type. If it is platform-specific, define a `ThreadResult` interface per platform and use it as the return type. This propagates type information to callers without casts.
- **Estimated scope**: ~5-8 sync service files (one per platform)

## Pattern 6: `Record<string, unknown>` for objects with known shapes (request bodies, metadata builders, API responses)

- **Seen in**: integrations-1 (message-sync body, project-management error maps), integrations-10 (customFieldsMap), integrations-11 (buildTaskMetadata return), integrations-12 (Facebook pages response), integrations-13 (stateMap), integrations-15 (Google Calendar event update body)
- **Category**: record-weakening
- **Combined impact**: medium -- at least 7 distinct sites use `Record<string, unknown>` or `Record<string, SomeType>` where the object's keys are statically known. This loses type safety for all field access and assignment.
- **What's happening**: Developers reach for `Record<string, unknown>` as a "flexible object" type when building request bodies, metadata objects, or lookup maps. In every case, the fields being assigned are known at write time, and SDK types or inline interfaces would provide full type checking.
- **Suggestion**: Replace each `Record<string, unknown>` with the appropriate typed alternative: SDK types for API request bodies (e.g., `Partial<calendar_v3.Schema$Event>`), named interfaces for metadata builders (e.g., `AsanaTaskMetadata`), and `as const satisfies` for lookup maps. Each replacement is local and low-risk.
- **Estimated scope**: ~10-15 sites across the integrations layer

## Pattern 7: `(await response.json()) as T` at HTTP client boundaries

- **Seen in**: integrations-10 (Asana client provider), integrations-12 (Facebook Messenger client provider), integrations-18 (Gmail fetch service with `z.object({}).loose() as ZodType<T>`), integrations-19 (Gmail sync-cache with 4 passthrough schemas)
- **Category**: type-cast
- **Combined impact**: low -- these casts occur at the JSON deserialization boundary where `fetch().json()` returns `any`. They are standard practice and largely unavoidable without runtime validation. The Gmail services use `z.object({}).loose() as ZodType<T>` as passthrough schemas, which is a pragmatic variant of the same pattern.
- **What's happening**: Every custom HTTP client provider has a generic `request<T>()` method that casts the JSON response to `T`. This is the only feasible approach without adding full Zod validation for every API response shape.
- **Suggestion**: No immediate action needed for the basic `as T` cast at the HTTP boundary. For the `z.object({}).loose() as ZodType<T>` passthrough pattern, extract a shared `passthroughSchema<T>()` helper to make intent explicit and reduce duplication. Long-term, consider adding optional Zod schema parameters to `request<T>()` for critical endpoints.
- **Estimated scope**: ~5-7 HTTP client providers + ~5 passthrough schema sites

## Pattern 8: Bracket-access on `Record<string, string>` identifiers in plugins

- **Seen in**: integrations-13 (GitHub plugin), integrations-16 (Google Drive plugin), integrations-18 (Google Gmail plugin)
- **Category**: missing-strict-typing
- **Combined impact**: medium -- at least 3 plugins access `params.parsed.identifiers["owner"]`, `["repo"]`, `["threadId"]`, etc. via string-keyed bracket access on `Record<string, string>`. If a key is absent, the access returns `undefined` at runtime but the type says `string`, which can cause downstream errors.
- **What's happening**: The base plugin framework types `identifiers` as `Record<string, string>`, meaning any key access compiles and is typed as `string` regardless of whether the key exists. Each platform needs specific keys (GitHub needs `owner`/`repo`/`number`, Gmail needs `threadId`, etc.).
- **Suggestion**: Define per-platform identifier interfaces (e.g., `GithubIdentifiers { owner: string; repo: string; number: string }`) and make the plugin base class generic on the identifiers type. Alternatively, enable `noUncheckedIndexedAccess` in tsconfig to make all `Record` index access return `T | undefined`.
- **Estimated scope**: ~8-10 plugin files + 1 base class change (or 1 tsconfig change for the `noUncheckedIndexedAccess` approach)
