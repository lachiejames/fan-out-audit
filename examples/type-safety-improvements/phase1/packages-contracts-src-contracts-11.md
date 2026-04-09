# Audit: packages-contracts-src-contracts-11

**Files inspected**: 8
**Findings**: 6

## Summary

This slice covers the Microsoft platform schemas (Calendar, OneDrive, Outlook, Teams) and the Microsoft Teams schemas file. The Teams expanded-data is notable for using `z.record(z.string(), teamsUserSchema)` for a `users` map (same pattern as Slack) — this is intentionally weak since user IDs are runtime strings. The Teams schemas have duplicate `chatType` enum definitions. The Microsoft metadata schemas are well-structured. The main issues are: `teamsReactionSchema` missing `.loose()` while the surrounding schemas all have it, inconsistent use of explicit interfaces for Teams types, and duplicate enum definitions.

## Findings

### Finding 1: Duplicate `chatType` enum in Teams schemas

- **File**: `packages/contracts/src/contracts/integrations/platforms/microsoft-teams/schemas.ts:15` and `packages/contracts/src/contracts/integrations/platforms/microsoft-teams/expanded-data.schema.ts:39`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `teamsChatTypeSchema` is defined in `schemas.ts` as `z.enum(["oneOnOne", "group", "meeting"])`. The exact same enum values appear inline in `teamsChatSchema` within `expanded-data.schema.ts` at line 39: `chatType: z.enum(["oneOnOne", "group", "meeting"])`. The inline version does not reuse the exported schema, meaning the two can diverge silently.
- **Suggestion**: Replace the inline `z.enum(["oneOnOne", "group", "meeting"])` in `teamsChatSchema` with the imported `teamsChatTypeSchema` from `schemas.ts`.
- **Evidence**:
  ```ts
  // schemas.ts line 15
  export const teamsChatTypeSchema = z.enum(["oneOnOne", "group", "meeting"]);
  // expanded-data.schema.ts line 39 — duplicate
  chatType: z.enum(["oneOnOne", "group", "meeting"]).optional(),
  ```

### Finding 2: `teamsReactionSchema` missing `.loose()` — inconsistent with all other Teams sub-schemas

- **File**: `packages/contracts/src/contracts/integrations/platforms/microsoft-teams/expanded-data.schema.ts:77`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: Every sub-schema in the Teams expanded data file (`teamsUserSchema`, `teamsChatSchema`, `teamsAttachmentSchema`, `teamsMessageBodySchema`, `teamsMessageSchema`) calls `.loose()` to allow SDK extra fields through. However `teamsReactionSchema` (line 77) is a plain `z.object(...)` without `.loose()`. If the Microsoft Graph SDK returns extra fields on reactions (e.g., `id` or `reactionContentUrl`), those would be silently stripped rather than passed through.
- **Suggestion**: Add `.loose()` to `teamsReactionSchema` to match the passthrough pattern used by all sibling schemas.
- **Evidence**: `export const teamsReactionSchema = z.object({ ... });` at line 77 — no `.loose()` call.

### Finding 3: Teams expanded-data types use `z.infer` — should use explicit interfaces

- **File**: `packages/contracts/src/contracts/integrations/platforms/microsoft-teams/expanded-data.schema.ts:133-136`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `TeamsUser`, `TeamsChat`, `TeamsMessage`, and `MicrosoftTeamsExpandedData` are all derived via `z.infer`. The CLAUDE.md explicit-interface pattern requires using explicit interfaces for expanded-data schemas participating in discriminated unions to prevent TS7056. While the Teams schema is not yet causing TS7056 errors, the schemas use `.loose()` extensively (which adds index signatures) — a combination known to trigger TS7056 in complex unions.
- **Suggestion**: Follow the Jira/Linear/Notion pattern: define explicit interfaces for `TeamsUser`, `TeamsChat`, `TeamsMessage`, and `MicrosoftTeamsExpandedData`, then bind schemas with `z.ZodType<Interface>` annotations.
- **Evidence**: `export type MicrosoftTeamsExpandedData = z.infer<typeof microsoftTeamsExpandedDataSchema>;` at line 136.

### Finding 4: `users: z.record(z.string(), teamsUserSchema)` — key type is unvalidated

- **File**: `packages/contracts/src/contracts/integrations/platforms/microsoft-teams/expanded-data.schema.ts:123`
- **Category**: record-weakening
- **Impact**: low
- **Description**: The `users` field in `microsoftTeamsExpandedDataSchema` is a `Record<string, TeamsUser>` keyed by user ID. This is identical to the Slack pattern (`slackExpandedDataSchema.users`). The key `string` type is necessary because user IDs are runtime values and cannot form a union type — this is an acceptable use of `Record<string, ...>`. However, a comment explaining this intentional design choice (rather than a stricter map) would improve maintainability.
- **Suggestion**: Add a comment on the `users` field: `// Keyed by Microsoft user ID (runtime string — cannot form a static union)`.
- **Evidence**: `users: z.record(z.string(), teamsUserSchema),` at line 123.

### Finding 5: `outlookExpandedDataSchema` and `gmailExpandedDataSchema` have identical shapes — no shared base

- **File**: `packages/contracts/src/contracts/integrations/platforms/microsoft-outlook/expanded-data.schema.ts:9` and `packages/contracts/src/contracts/integrations/platforms/google-gmail/expanded-data.schema.ts:12` (from slice 9)
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `outlookExpandedDataSchema` and `gmailExpandedDataSchema` have identical field sets (`anchorMessageId`, `pagination`, `platform`, `thread`). Both import from `emailThreadSchema`. The only difference is the `platform` literal. If the email schema ever gains a new field, both files must be updated independently.
- **Suggestion**: Create a shared `baseEmailExpandedDataSchema` in `core/email-expanded-data.schema.ts` with `anchorMessageId`, `pagination`, and `thread` fields, then extend it with the `platform` literal in each platform file.
- **Evidence**: Both files share `{ anchorMessageId: z.string().nullable(), pagination: paginationInfoSchema, platform: z.literal(...), thread: emailThreadSchema }`.

### Finding 6: OneDrive metadata `email: z.string()` should be `z.email()`

- **File**: `packages/contracts/src/contracts/integrations/platforms/microsoft-onedrive/platform-metadata.schema.ts:20`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `oneDriveMetadataSchema` uses `email: z.string()` for the user's Microsoft account email. Microsoft Graph returns a validated email address for this field. Using `z.email()` would validate format at deserialization time. This is the same issue noted in slice 9 for Google metadata schemas.
- **Suggestion**: Change `email: z.string()` to `email: z.email()` in `oneDriveMetadataSchema`, `outlookMetadataSchema`, and `microsoftCalendarMetadataSchema`.
- **Evidence**: `email: z.string(),` at line 20 in `microsoft-onedrive/platform-metadata.schema.ts`.
