# Audit: packages-contracts-src-contracts-9

**Files inspected**: 8
**Findings**: 5

## Summary

The Google platform files (Calendar, Docs, Drive, Gmail) share nearly identical schema shapes that differ only in their `platform` literal. The metadata schemas all use `.loose()` (Zod's passthrough) intentionally for SDK compatibility — that pattern is acceptable per the CLAUDE.md explicit-interface guidelines. The main concerns are: duplicate expanded-data shapes across Google Docs/Drive/OneDrive/Monday, the lack of `.strict()` on schemas where extra fields are unexpected, and the use of raw `z.string()` for email fields in metadata schemas where `z.email()` would catch malformed inputs at the contract boundary.

## Findings

### Finding 1: Duplicate expanded-data shape across Google Docs and Google Drive

- **File**: `packages/contracts/src/contracts/integrations/platforms/google-docs/expanded-data.schema.ts:3` and `packages/contracts/src/contracts/integrations/platforms/google-drive/expanded-data.schema.ts:3`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `googleDocsExpandedDataSchema` and `googleDriveExpandedDataSchema` have identical field sets (`platform`, `sourceUrl`, `summary`, `title`) differing only in the `platform` literal value. The same pattern also appears in `microsoft-onedrive` and `monday` (seen in slice 12). This creates four near-identical schemas with no shared base.
- **Suggestion**: Extract a shared `documentCardExpandedDataBaseSchema` with `sourceUrl`, `summary`, and `title` fields, then extend it with the platform literal in each file. This mirrors the pattern already used for email expanded data via `emailThreadSchema`.
- **Evidence**:
  ```ts
  // google-docs
  export const googleDocsExpandedDataSchema = z.object({
    platform: z.literal("google-docs"),
    sourceUrl: z.string().nullable(),
    summary: z.string().nullable(),
    title: z.string().nullable(),
  });
  // google-drive — identical shape, different literal
  export const googleDriveExpandedDataSchema = z.object({
    platform: z.literal("google-drive"),
    sourceUrl: z.string().nullable(),
    summary: z.string().nullable(),
    title: z.string().nullable(),
  });
  ```

### Finding 2: Raw `z.string()` for email field in GoogleCalendar/Gmail metadata schemas

- **File**: `packages/contracts/src/contracts/integrations/platforms/google-calendar/platform-metadata.schema.ts:12`, `packages/contracts/src/contracts/integrations/platforms/google-gmail/platform-metadata.schema.ts:13`, `packages/contracts/src/contracts/integrations/platforms/google-docs/platform-metadata.schema.ts:16`, `packages/contracts/src/contracts/integrations/platforms/google-drive/platform-metadata.schema.ts:16`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: All four Google metadata schemas declare `email: z.string()` rather than `z.email()`. The `email` field represents an OAuth-verified Google account email. Using `z.email()` would surface malformed data at deserialization time rather than silently passing bad data to the UI.
- **Suggestion**: Change `email: z.string()` to `email: z.email()` in all four Google metadata schemas. (Note: also applicable to Microsoft, LinkedIn, Slack, Monday, and Notion metadata schemas reviewed in later slices.)
- **Evidence**: `email: z.string(),` at line 12 in `google-calendar/platform-metadata.schema.ts`.

### Finding 3: `.loose()` on metadata schemas makes extra fields invisible to TypeScript but silently accepted

- **File**: `packages/contracts/src/contracts/integrations/platforms/google-calendar/platform-metadata.schema.ts:18`, all other `*-metadata.schema.ts` files in this slice
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: All metadata schemas call `.loose()` which adds `[key: string]: unknown` to the inferred type. This is intentional per the explicit-interface pattern, but the passthrough index signature is present even on the `GmailMetadata` / `GoogleCalendarMetadata` types via `PassthroughMetadata<...>`. Any typo in a metadata field name will silently pass runtime validation. This is a deliberate design trade-off (forward-compatible with new fields from Google APIs), but worth noting since stricter validation could use `.catchall(z.unknown())` to retain passthrough while being explicit.
- **Suggestion**: Document the intentional `.loose()` usage in a shared comment block so future contributors understand why strict mode is not used on metadata schemas.
- **Evidence**: `.loose()` at line 18 in `google-calendar/platform-metadata.schema.ts`.

### Finding 4: `GmailSyncMetaSchema` lacks a `strictStatusCodes`-equivalent validation — no `.strict()`

- **File**: `packages/contracts/src/contracts/integrations/platforms/google-gmail/schemas.ts:19`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `gmailSyncBodySchema` and `gmailSyncMetaSchema` are plain `z.object(...)` without `.strict()`. These are internal sync control schemas where extra fields would indicate a programming error (e.g., misspelled option names). Strict validation would catch such mistakes.
- **Suggestion**: Add `.strict()` to `gmailSyncBodySchema` and `gmailSyncMetaSchema` (and equivalently to `linearSyncBodySchema`, `slackSyncBodySchema` etc.) since callers always pass exact bodies with known shapes.
- **Evidence**: `export const gmailSyncMetaSchema = z.object({ ... });` at line 19.

### Finding 5: `gmailUserIdentitySchema` reused in GoogleCalendar but `googleUserIdentitySchema` used in Docs/Drive — inconsistent identity schema names

- **File**: `packages/contracts/src/contracts/integrations/platforms/google-calendar/platform-metadata.schema.ts:8`, `packages/contracts/src/contracts/integrations/platforms/google-docs/platform-metadata.schema.ts:9`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: Google Calendar imports `gmailUserIdentitySchema` (named after Gmail), while Google Docs and Google Drive import `googleUserIdentitySchema`. Two identity schemas exist for overlapping Google identity data. If they are structurally identical or near-identical, they should be consolidated into a single `googleUserIdentitySchema`.
- **Suggestion**: Verify whether `gmailUserIdentitySchema` and `googleUserIdentitySchema` have different shapes in `platform-metadata-shared.ts`. If identical, remove `gmailUserIdentitySchema` and update Google Calendar to use `googleUserIdentitySchema`.
- **Evidence**: `import { gmailUserIdentitySchema ... } from "../../platform-metadata-shared.ts"` (google-calendar, line 8) vs `import { googleUserIdentitySchema ... }` (google-docs, line 9).
