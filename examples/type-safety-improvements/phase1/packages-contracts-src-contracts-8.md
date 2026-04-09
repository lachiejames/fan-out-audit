# Audit: packages-contracts-src-contracts-8

**Files inspected**: 8
**Findings**: 8

## Summary

This slice covers the final integration platform schemas: the main platforms.ts (generated), and platform-specific expanded data and metadata schemas for Asana, Facebook Messenger, GitHub, and Google Calendar. The generated `platforms.ts` is excellent — fully typed with `satisfies Record<string, PublicPlatformConfig>`. The platform-specific schemas show a consistent pattern with `[key: string]: unknown` index signatures. Key issues: index signatures on every interface weaken property-level type safety, `GithubPrFile.status` is a loose string rather than an enum, and timestamps in platform schemas use `z.string()` instead of `isoDateTimeStringSchema`.

## Findings

### Finding 1: Index signature `[key: string]: unknown` on all Asana and GitHub expanded data interfaces weakens property access safety

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/platforms/asana/expanded-data.schema.ts:26` and `github/expanded-data.schema.ts:27`
- **Category**: any-usage
- **Impact**: medium
- **Description**: Every expanded data interface (e.g., `AsanaUser`, `AsanaProject`, `GithubUser`, `GithubComment`) includes `[key: string]: unknown`. This is required for the `.loose()` Zod schema pattern, but it means TypeScript will return `unknown` for any property access — even accesses to known, properly typed fields — unless the field is explicitly declared. It also prevents exhaustive checks on the interface. The CLAUDE.md documents this as intentional (TS7056 prevention), but the performance is a broad loss of type narrowing.
- **Suggestion**: The trade-off is documented. For types that appear in the discriminated union, the explicit interface with index signature is necessary. For leaf types (like `AsanaUser`, `GithubLabel`) that are NOT in the discriminated union, consider using `z.infer<typeof schema>` directly without the index signature to get full type safety.
- **Evidence**: `[key: string]: unknown;` in `AsanaUser` at line 26, `AsanaProject` at line 31, `GithubUser` at line 27, etc.

### Finding 2: `AsanaExpandedData.customFields` is `Record<string, unknown>` — opaque Asana custom fields

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/platforms/asana/expanded-data.schema.ts:97`
- **Category**: any-usage
- **Impact**: low
- **Description**: `customFields?: Record<string, unknown>` — same issue as in `content/schemas.ts`. Asana custom fields have a knowable shape from the Asana API but are left untyped.
- **Suggestion**: Define `AsanaCustomField = { display_value: string | null; name: string; type: string; [key: string]: unknown }` and use `Record<string, AsanaCustomField>`.
- **Evidence**: `customFields?: Record<string, unknown> | undefined;` at line 97 of `asana/expanded-data.schema.ts`.

### Finding 3: `GithubPrFile.status` is `z.string()` — should be an enum of GitHub statuses

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/platforms/github/expanded-data.schema.ts:73`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `GithubPrFile.status: string` with comment `// "added", "modified", "removed", "renamed"`. GitHub's API returns a fixed set of statuses for changed files. These are documented but not enforced.
- **Suggestion**: Change to `z.enum(["added", "modified", "removed", "renamed", "copied", "changed", "unchanged"])` and update the interface accordingly. The GitHub API docs list all valid values.
- **Evidence**: `status: z.string(), // "added", "modified", "removed", "renamed"` at line 177 of `github/expanded-data.schema.ts`.

### Finding 4: Timestamp fields in GitHub expanded data use `z.string()` without ISO datetime validation

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/platforms/github/expanded-data.schema.ts:134-222`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `createdAt`, `updatedAt`, `submittedAt` in GitHub schemas are all `z.string()` without ISO datetime format validation. Other contracts consistently use `isoDateTimeStringSchema`.
- **Suggestion**: Replace `z.string()` with `isoDateTimeStringSchema` for all timestamp fields. Note that GitHub returns timestamps in ISO 8601 format, so this is safe.
- **Evidence**: `createdAt: z.string(),` at multiple locations in `github/expanded-data.schema.ts`.

### Finding 5: `AsanaComment.createdAt` and `AsanaExpandedData.createdAt`/`updatedAt` use `z.string()` — same timestamp issue

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/platforms/asana/expanded-data.schema.ts:149-198`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: Asana timestamps (`createdAt`, `updatedAt`) are `z.string()` without datetime format validation.
- **Suggestion**: Replace with `isoDateTimeStringSchema`.
- **Evidence**: `createdAt: z.string(),` in `asanaCommentSchema` at line 152, and in `asanaExpandedDataSchema` at lines 180 and 195.

### Finding 6: `facebookMessengerMessageSchema.created_time` uses snake_case and `z.string()` — inconsistent with rest of codebase

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/platforms/facebook-messenger/expanded-data.schema.ts:31`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: The Facebook Messenger message schema uses `created_time` (snake_case) instead of `createdAt` (camelCase) and is a bare `z.string()`. This is likely reflecting the raw Meta Graph API field name. This inconsistency means consumers must remember different naming conventions per platform.
- **Suggestion**: Add a transform in the expand handler to normalize `created_time` to `createdAt`, and update the schema to use `createdAt: isoDateTimeStringSchema`. This is a backend mapping concern but the contract should reflect the normalized shape.
- **Evidence**: `created_time: z.string(),` at line 32 of `facebook-messenger/expanded-data.schema.ts`.

### Finding 7: `facebookMessengerMetadataSchema` fields `pageId` and `pageName` have no minimum length validation

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/platforms/facebook-messenger/platform-metadata.schema.ts:14-18`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `pageId: z.string()` and `pageName: z.string()` — both are required but can be empty strings. An empty `pageId` would cause silent failures in all Facebook Messenger API calls.
- **Suggestion**: Add `.min(1)` to both: `pageId: z.string().min(1)`, `pageName: z.string().min(1)`.
- **Evidence**: `pageId: z.string(),` and `pageName: z.string(),` at lines 14-15 of `facebook-messenger/platform-metadata.schema.ts`.

### Finding 8: `PLATFORMS` record key type is `string` on the `as const satisfies Record<string, PublicPlatformConfig>` — could use `PlatformId`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/core/generated/platforms.ts:1024`
- **Category**: record-weakening
- **Impact**: low
- **Description**: The generated `PLATFORMS` constant uses `satisfies Record<string, PublicPlatformConfig>`. The key type is `string`. Once `PlatformId` is derived as `keyof typeof PLATFORMS`, type-safe access works. However, any function receiving `Record<string, PublicPlatformConfig>` loses the `PlatformId` key constraint. This is a read-only access pattern issue (consumers usually import `PLATFORMS` directly), so impact is low.
- **Suggestion**: This is cosmetic given that `PlatformId = keyof typeof PLATFORMS` is the canonical type. The constraint is appropriate as-is. No change needed unless a future refactor passes the `PLATFORMS` record itself as a function argument.
- **Evidence**: `} as const satisfies Record<string, PublicPlatformConfig>;` at line 1024.
