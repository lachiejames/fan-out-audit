# Cross-Cutting Patterns (Group 8)

**Slices analyzed**: apps-api-src-integrations-30, apps-api-src-integrations-31, apps-api-src-integrations-32, apps-api-src-integrations-33, apps-api-src-integrations-34, apps-api-src-integrations-4, apps-api-src-integrations-5, apps-api-src-integrations-6, apps-api-src-integrations-7, apps-api-src-integrations-8, apps-api-src-integrations-9, apps-api-src-interface-1

## Pattern 1: `BaseSyncService` uses `unknown[]` for parsedItems, forcing casts in every platform override

- **Seen in**: apps-api-src-integrations-31 (Notion sync), apps-api-src-integrations-33 (Slack sync)
- **Category**: type-cast
- **Combined impact**: high -- every platform sync service that extends `BaseSyncService` must cast `parsedItems` from `unknown[]` to their platform-specific type (e.g., `ParsedNotionItem[]`, `MappedSlackMessage[]`). This affects every current and future integration.
- **What's happening**: `BaseSyncService` uses a template method pattern where `parsedItems` is typed as `unknown[]`. Each platform override (Notion, Slack, and presumably Gmail, Linear, etc.) must perform an unsafe `as PlatformItem[]` cast at the usage site.
- **Suggestion**: Make `BaseSyncService` generic: `class BaseSyncService<TItem>` with `parsedItems: TItem[]` and `syncAttachments(parsedItems: TItem[])`. Each platform subclass parameterizes with its own item type, eliminating all casts.
- **Estimated scope**: 1 base class change + ~6-10 platform sync service files

## Pattern 2: `platformMetadata as PlatformMetadata | null` -- unvalidated cast from DB `unknown` column

- **Seen in**: apps-api-src-integrations-30 (Notion OAuth), apps-api-src-integrations-32 (Slack OAuth)
- **Category**: type-cast
- **Combined impact**: high -- every platform's OAuth service casts `integration.platformMetadata` (stored as `unknown`/JSONB in DB) to its platform-specific metadata type without runtime validation. A schema migration or corrupt row would silently produce wrong data.
- **What's happening**: The `integrations` table stores `platformMetadata` as an untyped JSON column. Each OAuth service reads it and casts directly: `integration.platformMetadata as SlackPlatformMetadata | null`, `integration.platformMetadata as NotionPlatformMetadata | null`.
- **Suggestion**: Add a per-platform `parsePlatformMetadata(raw: unknown): PlatformMetadata | null` function using Zod. A shared factory `createMetadataParser(schema)` would standardize this across all platforms.
- **Estimated scope**: ~10-12 OAuth service files (one per platform)

## Pattern 3: `if ("code" in error) return error as SlackError` -- repeated unsafe error cast in mapErr callbacks

- **Seen in**: apps-api-src-integrations-32 (slack-huddle, slack-message-actions, slack-message-read, slack-search)
- **Category**: type-cast
- **Combined impact**: medium -- 5+ occurrences of the identical unsafe cast pattern across Slack services. Checking for a single `code` property is insufficient to confirm the error matches the `SlackError` discriminated union.
- **What's happening**: Multiple Slack services use `.mapErr` callbacks that check `"code" in error` and then cast to `SlackError` without verifying the full discriminant shape. If any non-Slack error happens to have a `code` field, it would be mistyped.
- **Suggestion**: Create a shared `isSlackError(e: unknown): e is SlackError` type guard that validates the discriminant field(s) against the `SlackError` union. Replace all 5+ occurrences with `if (isSlackError(error)) return error`.
- **Estimated scope**: 1 new type guard + 5-6 service files updated

## Pattern 4: Notion SDK union return types cast without type guards (GetPageResponse, GetDatabaseResponse, SearchResponse)

- **Seen in**: apps-api-src-integrations-30 (notion-fetch, notion-search, notion.plugin), apps-api-src-integrations-31 (notion-sync, notion-type-mapper)
- **Category**: type-cast
- **Combined impact**: high -- 6+ casts across Notion services to narrow SDK union types. The Notion SDK returns union types (e.g., `PageObjectResponse | PartialPageObjectResponse`) and the codebase consistently casts instead of narrowing.
- **What's happening**: `pages.retrieve()` returns `GetPageResponse` (union), `databases.retrieve()` returns `GetDatabaseResponse` (union), `search()` returns a union of page/database results. Each call site casts to the full variant (`as PageObjectResponse`, `as { description?: ...; icon?: ... }`, `as { id: string; icon?: ... }`) without checking which variant was returned.
- **Suggestion**: Create a shared `notion-type-guards.utils.ts` with: `isFullPage(r: GetPageResponse): r is PageObjectResponse`, `isFullDatabase(r: GetDatabaseResponse): r is DatabaseObjectResponse`, `isDatabaseSearchResult(r): r is DatabaseSearchResult`. Replace all casts with guard checks.
- **Estimated scope**: 1 new utils file + 4-5 service/util files updated

## Pattern 5: `Record<string, unknown>` for structured data with known shapes (cursor state, column values, filter params, event payloads)

- **Seen in**: apps-api-src-integrations-4 (tiered engine, plan.types), apps-api-src-integrations-7 (sync-progress-pubsub), apps-api-src-integrations-8 (cursor-state utils), apps-api-src-integrations-30 (monday-write, notion-search), apps-api-src-integrations-34 (integrations.service)
- **Category**: record-weakening
- **Combined impact**: medium -- at least 8 distinct instances across the integrations layer where `Record<string, unknown>` is used for data with a known or partially known shape. This pattern cascades: accepting `Record<string, unknown>` forces downstream consumers to cast individual fields.
- **What's happening**: Several categories of data share this pattern: (a) cursor state (`{ v: 1, tierIndex, offset }` stored as `Record<string, unknown>`), (b) Monday column values, (c) Notion database filter params, (d) sync progress event payloads with known `platform`/`userId` fields, (e) DB update objects with known column names.
- **Suggestion**: Replace each with its appropriate narrower type: versioned discriminated union for cursor state, `QueryDatabaseParameters["filter"]` for Notion filters, `Partial<typeof integrationsTable.$inferInsert>` for DB updates, `{ platform: string; userId: string; [key: string]: unknown }` for event payloads.
- **Estimated scope**: ~8-10 files, each requiring a small type definition + usage update

## Pattern 6: OAuth/wizard layer uses pervasive `unknown` to avoid threading generics, causing cast chains

- **Seen in**: apps-api-src-integrations-5 (base-sync-lifecycle, base-sync), apps-api-src-integrations-6 (oauth-base.wizard, oauth-base.service)
- **Category**: type-cast, missing-strict-typing
- **Combined impact**: medium -- the `WizardDeps` interface and `OAuthBaseService.handleCallback` use `unknown` for `tokenData`, `userInfo`, and integration query results. Every consumer must cast these back to concrete types, creating a chain of unsafe casts.
- **What's happening**: `WizardDeps` deliberately uses `unknown` to avoid threading `TTokenResponse`/`TUserInfo` generics. The comment "Duck-typed to avoid threading generics" explains the intent. But this means every call site in `oauth-base.wizard.ts` and `oauth-base.service.ts` must cast: `tokenData as TTokenResponse`, `existingResult.value as { id: string; ... }`, etc.
- **Suggestion**: Define a minimal `OAuthTokenFields` interface (`{ accessToken: string; expiresIn: number; refreshToken: string | null }`) that captures what the wizard actually needs, replacing `unknown` with this narrower but still platform-agnostic type. For integration lookups, define `MinimalIntegration = { id: string; platform: PlatformId; platformUserId?: string | null }`.
- **Estimated scope**: 2-3 core files (wizard, oauth-base.service, callback deps type)

## Pattern 7: Vercel AI SDK `generateText` result accessed via defensive checks and casts instead of typed properties

- **Seen in**: apps-api-src-integrations-6 (query-expansion.service, reply-classifier.service)
- **Category**: type-cast, missing-strict-typing
- **Combined impact**: low -- 2 instances of identical boilerplate. The `generateText` return type includes a typed `usage` field, but the code uses `typeof result === "object" && "usage" in result ? (result as { usage?: unknown }).usage : undefined` instead of `result.usage`.
- **What's happening**: Both services extract `usage` from `generateText` results using a defensive runtime check + cast pattern, even though the Vercel AI SDK types the return value with a `usage` property.
- **Suggestion**: Replace with direct `result.usage` access. If the defensive check was added for a historical SDK version, it is no longer needed.
- **Estimated scope**: 2 files, 1 line each

## Pattern 8: DB string columns cast to narrower union types without runtime validation

- **Seen in**: apps-api-src-integrations-5 (files-api.service -- `purpose as FilePurpose`), apps-api-src-integrations-8 (search-candidate-mapping -- `contentLevel as "message" | "thread"`), apps-api-src-integrations-5 (base-sync-lifecycle -- `phase as Parameters<...>[1]`)
- **Category**: type-cast
- **Combined impact**: medium -- at least 3 distinct instances where a DB `text` column value is cast to a TypeScript union without validation. If a new enum value is added to the DB but not to the union, the cast silently produces an invalid value.
- **What's happening**: Drizzle returns `string` for text columns. The codebase casts these to narrower unions (`FilePurpose`, `"message" | "thread"`, phase unions) at the point of use without any runtime check.
- **Suggestion**: Create a shared utility `parseEnum<T extends string>(value: string, validValues: readonly T[]): T | null` or use Zod enum parsing at the DB boundary. Apply consistently wherever DB strings are narrowed to unions.
- **Estimated scope**: ~5-8 files across the integrations layer, potentially more across the full codebase
