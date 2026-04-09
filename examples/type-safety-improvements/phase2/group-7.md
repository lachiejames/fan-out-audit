# Cross-Cutting Patterns (Group 7)

**Slices analyzed**: apps-api-src-integrations-2, apps-api-src-integrations-3, apps-api-src-integrations-20, apps-api-src-integrations-21, apps-api-src-integrations-22, apps-api-src-integrations-23, apps-api-src-integrations-24, apps-api-src-integrations-25, apps-api-src-integrations-26, apps-api-src-integrations-27, apps-api-src-integrations-28, apps-api-src-integrations-29

## Pattern 1: `BaseSyncService` uses `unknown[]` for parsed items, forcing casts in every subclass

- **Seen in**: integrations-21 (Jira), integrations-23 (Linear), integrations-25 (OneDrive), integrations-28 (Monday)
- **Category**: type-cast
- **Combined impact**: High -- every sync service subclass must cast `parsedItems as ConcreteType[]` in `getEmbeddingTexts` and `onContentInserted` overrides, creating an unchecked type hole replicated across all platforms
- **What's happening**: `BaseSyncService` declares abstract methods with `parsedItems: unknown[]`. Each concrete sync service (Jira, Linear, OneDrive, Monday, Outlook, Microsoft Calendar, Teams) immediately casts to its platform-specific item type (e.g., `ResolvedIssue[]`, `MondayItem[]`) without runtime validation.
- **Suggestion**: Add a generic type parameter to `BaseSyncService<TItem>` so subclasses declare their item type once (`JiraSyncService extends BaseSyncService<ResolvedIssue>`) and overrides receive `parsedItems: TItem[]` without any cast.
- **Estimated scope**: 1 base class change + 7-8 sync service subclasses

## Pattern 2: `.filter(Boolean) as T[]` instead of typed filter predicates

- **Seen in**: integrations-21 (Jira), integrations-23 (Linear), integrations-24 (Microsoft Calendar), integrations-25 (OneDrive), integrations-28 (Monday)
- **Category**: type-cast
- **Combined impact**: Medium -- the pattern is widespread but each instance is individually low-risk; the cast could hide bugs if the cache `.get()` return type changes
- **What's happening**: After mapping over a cache with `.map(id => cache.get(id))`, the code uses `.filter(Boolean) as ConcreteType[]` to remove `undefined` entries. TypeScript does not narrow through `Boolean` as a filter predicate, so the cast is required.
- **Suggestion**: Replace with a typed filter predicate: `.filter((x): x is ConcreteType => x !== undefined)`. Or define a shared utility `function isDefined<T>(x: T | undefined): x is T` and use `.filter(isDefined)` across all sync services.
- **Estimated scope**: 6-8 instances across sync services

## Pattern 3: `parseCursorState()` output cast to platform-specific cursor type without validation

- **Seen in**: integrations-22 (Linear), integrations-24 (Microsoft Calendar), integrations-25 (OneDrive), integrations-26 (Outlook, implied), integrations-27 (Teams, implied), integrations-28 (Monday, Teams explicit)
- **Category**: type-cast
- **Combined impact**: Medium -- `parseCursorState` returns `unknown` and every plugin casts to its own cursor state type (e.g., `as LinearCursorState`, `as MicrosoftCalendarCursorState`, `as OneDriveCursorState`, `as MicrosoftTeamsCursorState`). If the stored cursor is from a prior schema version, the cast succeeds silently and causes runtime failures.
- **What's happening**: Each plugin file calls `parseCursorState({ value: params.resumeState })` and immediately casts to its platform-specific cursor type. There is no Zod schema validation at the boundary.
- **Suggestion**: Create a generic helper `parseCursorStateAs<T>({ value, schema }: { value: unknown; schema: z.ZodType<T> }): T` that validates before returning. Each platform defines a Zod schema for its cursor state.
- **Estimated scope**: 6-7 plugin files, 1 new shared utility

## Pattern 4: `nextResumeState as PlatformCursorState` cast on locally-built objects

- **Seen in**: integrations-21 (Jira), integrations-23 (Linear), integrations-28 (Monday)
- **Category**: type-cast
- **Combined impact**: Low-Medium -- the objects are locally constructed so the risk of shape mismatch is lower than external data casts, but `satisfies` would provide compile-time validation at the construction site
- **What's happening**: Sync services build a `nextResumeState` object from local variables and then cast it `as PlatformCursorState` to assign to `ctx.resumeState`. The object literal is not structurally validated against the cursor type.
- **Suggestion**: Use `satisfies PlatformCursorState` on the object literal instead of `as PlatformCursorState` on assignment. This gives compile-time validation without changing runtime behavior.
- **Estimated scope**: 5-6 sync service files

## Pattern 5: `platformMetadata as PlatformMetadataType | null` cast from `unknown` without runtime validation

- **Seen in**: integrations-20 (Google), integrations-21 (Jira), integrations-23 (LinkedIn), integrations-26 (Outlook), integrations-28 (Monday)
- **Category**: type-cast
- **Combined impact**: High -- `platformMetadata` comes from JSONB database columns typed as `unknown`. Every OAuth service and some plugin files cast it to their platform-specific metadata type without Zod validation. If the stored JSON has drifted from the current schema (e.g., after a migration), the cast succeeds and downstream property access returns `undefined` or throws.
- **What's happening**: The `buildMasterUserInfo`, `buildMasterTokenData`, `extractSyncConfig`, and plugin methods all receive `integration.platformMetadata` typed as `unknown` and immediately cast to platform-specific metadata types like `JiraPlatformMetadata`, `GooglePlatformMetadata`, `LinkedinPlatformMetadata`, or `Record<string, unknown>`.
- **Suggestion**: Define a Zod schema per platform metadata type and validate at the boundary (i.e., when reading from the database row). A shared base method in the OAuth base service could handle parsing: `protected parsePlatformMetadata(raw: unknown): Result<TPlatformMetadata, ValidationError>`.
- **Estimated scope**: 8-10 OAuth/plugin service files across all platforms

## Pattern 6: Microsoft platform metadata interfaces with redundant `extends Record<string, unknown>` and `[key: string]: unknown`

- **Seen in**: integrations-25 (OneDrive), integrations-26 (Outlook), integrations-27 (Teams)
- **Category**: record-weakening
- **Combined impact**: Medium -- the redundant index signatures defeat exhaustive property checking on all three Microsoft platform metadata types, and the pattern is copy-pasted identically across OneDrive, Outlook, and Teams OAuth services
- **What's happening**: `MicrosoftOneDrivePlatformMetadata`, `MicrosoftOutlookPlatformMetadata`, and `MicrosoftTeamsPlatformMetadata` all extend `MicrosoftPlatformMetadata<T>` AND `Record<string, unknown>`, then also declare `[key: string]: unknown`. This double index signature is redundant and prevents TypeScript from flagging typos or invalid property access.
- **Suggestion**: Remove `extends Record<string, unknown>` and the `[key: string]: unknown` line from all three interfaces. If the parent `MicrosoftPlatformMetadata` or `PlatformMetadata` already needs an index signature, it should be addressed once at the root type.
- **Estimated scope**: 3 files (microsoft-onedrive-oauth.service.ts, microsoft-outlook-oauth.service.ts, microsoft-teams-oauth.service.ts)

## Pattern 7: HTTP API client methods cast `response.json() as T` without validation

- **Seen in**: integrations-21 (Jira client), integrations-22 (Linear OAuth GraphQL), integrations-23 (LinkedIn client), integrations-28 (Monday GraphQL client)
- **Category**: type-cast
- **Combined impact**: High -- every platform API client's generic `get<T>`/`post<T>`/`query<T>` method casts the raw JSON response to the caller's requested type. This is the most fundamental type hole in the integration layer: a malformed API response, CDN error page, or rate-limit JSON body will silently satisfy the cast and propagate incorrect data.
- **What's happening**: `JiraApiClient.get<T>()`, `LinkedinApiClient.get()` (returns `unknown`), `MondayApiClient.query<T>()`, and Linear's raw `response.json()` calls all cast without Zod validation. LinkedIn is slightly different (returns `Promise<Result<unknown, ...>>`) but still pushes validation to callers.
- **Suggestion**: Add a `schema: z.ZodType<T>` parameter to each client method (e.g., `get<T>({ path, schema }: { path: string; schema: z.ZodType<T> }): Promise<Result<T, PlatformError>>`) and validate with `schema.parse()` before returning. This is a large refactor but eliminates the single biggest category of type casts in the integration layer.
- **Estimated scope**: 4 client provider files + all call sites (likely 30-50 endpoint invocations)

## Pattern 8: Unnecessary `as Date` cast on `calculateTokenExpiry` wrappers

- **Seen in**: integrations-20 (Google), integrations-23 (Linear)
- **Category**: type-cast
- **Combined impact**: Low -- only 2 instances, trivial fix
- **What's happening**: Both `google-oauth.utils.ts` and `linear-oauth.utils.ts` wrap a shared `_calculateTokenExpiry` function and cast the return to `Date`, even though `_calculateTokenExpiry` already declares `Date` as its return type.
- **Suggestion**: Remove the `as Date` cast in both files.
- **Estimated scope**: 2 files, 1 line each

## Pattern 9: `[key: string]: unknown` index signatures on platform-specific types defeating exhaustive checking

- **Seen in**: integrations-20 (GooglePlatformMetadata), integrations-22 (JiraApiIssueFields), integrations-25 (OneDrive metadata), integrations-26 (Outlook metadata), integrations-27 (Teams metadata)
- **Category**: record-weakening
- **Combined impact**: Medium -- index signatures on types that also have named properties prevent TypeScript from flagging misspelled property names and allow arbitrary string-key access. This is a broader version of Pattern 6 that extends beyond Microsoft platforms.
- **What's happening**: Multiple platform types define named properties alongside an open `[key: string]: unknown` index signature. For `JiraApiIssueFields`, the motivation is custom fields; for metadata types, it appears to be historical compatibility.
- **Suggestion**: For types where dynamic keys are genuinely needed (e.g., Jira custom fields), split into `knownFields: { summary: string; ... }` and `customFields: Record<string, unknown>` as separate properties. For metadata types, remove the index signature entirely.
- **Estimated scope**: 5-6 type definitions across platforms

## Pattern 10: `fetchThread` / `fetchOnDemandFile` returning `Promise<unknown>` despite building well-typed objects

- **Seen in**: integrations-25 (OneDrive `fetchOnDemandFile`), integrations-28 (Monday `fetchThread`)
- **Category**: missing-strict-typing
- **Combined impact**: Medium -- these functions construct fully-known object shapes internally but declare `Promise<unknown>` return types, forcing every caller to cast or validate. The type information exists at the construction site but is thrown away.
- **What's happening**: Both functions build result objects with known properties (`id`, `name`, `content`, etc.) but return `Promise<unknown>`, pushing type unsafety to callers.
- **Suggestion**: Define interfaces (`OnDemandFileResult`, `MondayBoardFetchResult`) matching the returned object shape and use them as return types.
- **Estimated scope**: 2 files + their call sites

## Pattern 11: Weak Zod schemas using `z.custom<SdkType>(isRecord)` that accept any object

- **Seen in**: integrations-22 (Linear `linearViewerSchema`, `linearIssueSchema`), integrations-27 (Teams `microsoftTeamsItemSchema`)
- **Category**: missing-strict-typing
- **Combined impact**: Medium -- cache validation schemas that accept any non-null object provide zero protection against corrupted or stale cache entries. A cache entry missing required fields (e.g., `id`) will pass validation and cause downstream runtime errors.
- **What's happening**: Zod schemas defined as `z.custom<SdkType>(isRecord)` where `isRecord` only checks `typeof value === "object" && value !== null`, or `z.object({}).loose()`. These schemas validate to the SDK type but do not check any fields.
- **Suggestion**: At minimum, validate the `id` field: `z.object({ id: z.string() }).passthrough()`. For SDK types with `__typename`, validate that as well.
- **Estimated scope**: 3-4 schema definitions across Linear and Teams

## Pattern 12: Duplicate `BudgetConfig` type alias

- **Seen in**: integrations-2 (both `base-integration-plugin.ts` and `integration-plugin.interface.ts`)
- **Category**: duplicate-type
- **Combined impact**: Low -- 2 files re-export the same type alias `BudgetConfig = PlatformBudgetConfig`
- **What's happening**: Two files in the core integrations layer independently alias `PlatformBudgetConfig` as `BudgetConfig`, creating parallel re-exports of the same type.
- **Suggestion**: Remove both aliases. Callers should import `PlatformBudgetConfig` directly from `@slopweaver/contracts`.
- **Estimated scope**: 2 files + update import sites
