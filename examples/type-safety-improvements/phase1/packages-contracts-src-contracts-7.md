# Audit: packages-contracts-src-contracts-7

**Files inspected**: 8
**Findings**: 8

## Summary

This slice covers the core integration schemas, sync types, workspace search, pending schemas, platform config types, platform metadata shared, and platform types. The platform config and platform types files are strong with proper union types. Key findings: several `z.string()` fields in metadata schemas where enum or datetime validators should be used, `WorkItemConfig.type` is an untyped string rather than a `WorkItemType` union, and `slackMessageMetadataSchema` duplicates fields from the content-level `slackContentMetadataSchema` in contracts-3.

## Findings

### Finding 1: `slackMessageMetadataSchema` in `core/schemas.ts` duplicates `slackContentMetadataSchema` from `content/schemas.ts`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/core/schemas.ts:64-69`
- **Category**: duplicate-type
- **Impact**: high
- **Description**: `slackMessageMetadataSchema` (in `core/schemas.ts`) and `slackContentMetadataSchema` (in `content/schemas.ts`) are near-identical: both have `channel: z.string()`, `user: z.string().optional()`, `materialized: z.boolean().optional()`. The core schema adds `channelName` and `messageCount` that the content schema doesn't have. Two schemas for "Slack metadata" with overlapping but slightly different shapes creates drift risk.
- **Suggestion**: Consolidate into a single canonical Slack metadata schema (likely the richer one from `content/schemas.ts`) and extend or omit fields as needed in the two usage contexts. If the distinction is intentional, add a comment explaining the difference.
- **Evidence**: `slackMessageMetadataSchema` at lines 64-69 in `core/schemas.ts` vs `slackContentMetadataSchema` in `content/schemas.ts` lines 276-281.

### Finding 2: `gmailMessageMetadataSchema` duplicates `gmailContentMetadataSchema` similarly

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/core/schemas.ts:77-84`
- **Category**: duplicate-type
- **Impact**: high
- **Description**: `gmailMessageMetadataSchema` and `gmailContentMetadataSchema` both model Gmail message metadata with overlapping fields (`cc`, `from`, `to`, `labels`). The core schema adds `messageCount` for thread context; the content schema adds `snippet` and `materialized`. Again two schemas for the same underlying concept.
- **Suggestion**: Same as Finding 1 — consolidate with a shared base and extend for context-specific needs.
- **Evidence**: `gmailMessageMetadataSchema` at lines 77-84 in `core/schemas.ts`.

### Finding 3: `linearMessageMetadataSchema.priority` has no range constraint

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/core/schemas.ts:95`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `priority: z.number()` — comment says "0-4" but no `.min(0).max(4)` constraint. Linear's priority enum is 0 (None), 1 (Urgent), 2 (High), 3 (Medium), 4 (Low).
- **Suggestion**: `priority: z.number().int().min(0).max(4)` or better, `z.enum([0, 1, 2, 3, 4])` to make the discrete values explicit.
- **Evidence**: `priority: z.number(), // 0-4` at line 95 of `core/schemas.ts`.

### Finding 4: `WorkItemConfig.type` is `string` in `platform-config.ts` — should reference `WorkItemType`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/platform-config.ts:143`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `WorkItemConfig.type: string` accepts any action type string. The codebase has a `workItemTypeSchema` / `WorkItemType` union in the work-items contract. The generated `PLATFORMS` constant also has `workItems[n].type` as a literal string. Without the type constraint, a typo in a config file would only be caught at runtime.
- **Suggestion**: Import `WorkItemType` from the work-items contract and type this as `type: WorkItemType`. The generated `PLATFORMS` constant is validated with `satisfies Record<string, PublicPlatformConfig>`, so strengthening this interface will propagate the check to all config files.
- **Evidence**: `type: string; // Action type identifier` at line 143 of `platform-config.ts`.

### Finding 5: `syncDiagnosticsSchema` timestamp fields use `z.string()` without datetime format

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/platform-metadata-shared.ts:26-42`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `syncDiagnosticsSchema.lastSyncResult.timestamp`, `lastError.timestamp` — both `z.string()` without ISO datetime validation. Diagnostics are displayed in the UI; an invalid timestamp string would render incorrectly.
- **Suggestion**: Use `isoDateTimeStringSchema` for these fields.
- **Evidence**: `timestamp: z.string(),` appears twice in `syncDiagnosticsSchema`.

### Finding 6: `teamsMessageMetadataSchema.chatType` uses `z.enum(["oneOnOne", "group", "meeting"]).optional()` — good, but `webUrl` is unvalidated

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/core/schemas.ts:118-123`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `webUrl: z.string().optional()` — the field is a URL and should use `z.url()` for format validation. Other URL fields in the codebase consistently use `z.url()`.
- **Suggestion**: Change to `webUrl: z.url().optional()`.
- **Evidence**: `webUrl: z.string().optional(),` at line 122.

### Finding 7: `OAuthConfig.scopesByAccessMode` uses `Partial<Record<OAuthAccessMode, string[]>>` — all modes could be absent

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/platform-config.ts:31`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `scopesByAccessMode: Partial<Record<OAuthAccessMode, string[]>>` allows all access modes to have no scopes. If the code that reads scopes for a given access mode uses `scopesByAccessMode[mode]!` without checking, it would produce undefined.
- **Suggestion**: If every platform must have at least "read" scopes, use `{ read: string[] } & Partial<Record<OAuthAccessMode, string[]>>` to enforce the minimum. Alternatively enforce in the config-level schema.
- **Evidence**: `scopesByAccessMode: Partial<Record<OAuthAccessMode, string[]>>;` at line 31 of `platform-config.ts`.

### Finding 8: `pendingPlatformMetadataSchema` in `pending/schemas.ts` uses `.loose()` — same issue noted in contracts-5, but worth repeating the scope

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/pending/schemas.ts:16-19`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: Already noted in contracts-5 Finding 4 — including here for completeness since this file is in the contracts-7 slice.
- **Suggestion**: See contracts-5 Finding 4 suggestion.
- **Evidence**: `.loose()` on `pendingPlatformMetadataSchema`.
