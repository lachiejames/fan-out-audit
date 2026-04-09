# Audit: packages-contracts-src-contracts-12

**Files inspected**: 8
**Findings**: 6

## Summary

This slice covers Monday, Notion, Slack, and their constants/schemas files. Notion's expanded-data is the most complex file reviewed, using the explicit-interface pattern correctly. The main issues are: Slack's `attachments: z.array(z.unknown())` on `slackMessageSchema`, the `users: z.record(z.string(), slackUserSchema)` pattern (same as Teams), `slackBlockElementSchema` using `z.string()` for its `type` field where a Slack-defined enum exists, and the `mondayExpandedDataSchema` being a duplicate of the Google Docs/Drive shape.

## Findings

### Finding 1: `slackMessageSchema.attachments` uses `z.array(z.unknown())` — typed schema exists

- **File**: `packages/contracts/src/contracts/integrations/platforms/slack/expanded-data.schema.ts:174`
- **Category**: any-usage
- **Impact**: medium
- **Description**: `slackMessageSchema` uses `attachments: z.array(z.unknown()).optional()` for Slack message attachments. Slack's API has a documented attachment shape (`fallback`, `color`, `pretext`, `author_name`, `title`, `text`, `fields`, `footer`). Unlike `blocks` (which does have a typed `slackBlockSchema`), attachments receive no type safety at all, making any downstream attachment rendering code rely on runtime checks.
- **Suggestion**: Define a `slackAttachmentSchema` (similar to the existing `slackFileSchema` and `slackBlockSchema`) with at minimum `fallback?: string`, `color?: string`, `pretext?: string`, `text?: string`, and `title?: string`, using `.loose()` for forward compatibility.
- **Evidence**: `attachments: z.array(z.unknown()).optional(),` at line 174.

### Finding 2: `slackBlockElementSchema.type` uses `z.string()` — Slack element type enum is well-known

- **File**: `packages/contracts/src/contracts/integrations/platforms/slack/expanded-data.schema.ts:151`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `slackBlockElementSchema` uses `type: z.string()` for the block element type. Slack documents a fixed set of element types: `"button"`, `"checkboxes"`, `"datepicker"`, `"image"`, `"multi_static_select"`, `"overflow"`, `"plain_text_input"`, `"radio_buttons"`, `"static_select"`, `"timepicker"`, `"users_select"`, `"conversations_select"`, `"channels_select"`, `"external_select"`, `"rich_text_section"`, `"rich_text_list"`, `"rich_text_preformatted"`, `"rich_text_quote"`. A `z.string()` type prevents exhaustive handling in the frontend.
- **Suggestion**: At minimum, use a non-exhaustive union of known types with a `.or(z.string())` fallback, e.g., `z.enum(["button", "checkboxes", "image", ...]).or(z.string())`. This preserves forward compatibility while improving IDE autocomplete and documentation.
- **Evidence**: `type: z.string(),` at line 151.

### Finding 3: `mondayExpandedDataSchema` is a fourth duplicate of the document-card shape

- **File**: `packages/contracts/src/contracts/integrations/platforms/monday/expanded-data.schema.ts:3`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `mondayExpandedDataSchema` (`platform`, `sourceUrl`, `summary`, `title`) is the fourth identical schema shape after `googleDocsExpandedDataSchema`, `googleDriveExpandedDataSchema`, and `oneDriveExpandedDataSchema`. All four schemas are structurally identical except for their `platform` literal.
- **Suggestion**: Extract `documentCardExpandedDataBaseSchema = z.object({ sourceUrl: z.string().nullable(), summary: z.string().nullable(), title: z.string().nullable() })` in `core/` and have each platform schema extend it with `platform: z.literal("...")`.
- **Evidence**: `export const mondayExpandedDataSchema = z.object({ platform: z.literal("monday"), sourceUrl: ..., summary: ..., title: ... });` matches the three Google/OneDrive schemas exactly.

### Finding 4: `NotionPage.properties` and `NotionDatabase.properties` use `Record<string, NotionPropertyValue>` — key is untyped

- **File**: `packages/contracts/src/contracts/integrations/platforms/notion/expanded-data.schema.ts:164` and `203`
- **Category**: record-weakening
- **Impact**: low
- **Description**: Both `NotionPage.properties` and `NotionDatabase.properties` are typed as `Record<string, NotionPropertyValue>`. Notion property keys are user-defined field names (e.g., `"Status"`, `"Assignee"`). Since these are runtime strings, `Record<string, ...>` is correct and cannot be made more specific without knowing the database schema. However, the Zod schema uses `z.record(z.string(), notionPropertyValueSchema)` which validates each value but not the key format.
- **Suggestion**: This is an intentional design decision. Add a comment on both `properties` fields: `// Keys are user-defined property names — cannot form a static union`. No code change required.
- **Evidence**: `properties: Record<string, NotionPropertyValue>;` at lines 164 and 203.

### Finding 5: `notionMetadataSchema.userIdentity` uses an inline anonymous object schema

- **File**: `packages/contracts/src/contracts/integrations/platforms/notion/platform-metadata.schema.ts:14`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `notionMetadataSchema` defines `userIdentity` as an anonymous inline `z.object({ name: z.string().nullable(), userId: z.string() }).optional()`. Unlike Jira, Monday, Linear, and other platforms that export a named `*UserIdentitySchema`, Notion's identity schema is anonymous and cannot be imported or reused.
- **Suggestion**: Extract `notionUserIdentitySchema = z.object({ name: z.string().nullable(), userId: z.string() })` and export it, then reference it in `notionMetadataSchema`. Add `.strict()` since the fields are well-known (Notion API `bot` object).
- **Evidence**: Inline `z.object({ name: z.string().nullable(), userId: z.string() }).optional()` at line 14.

### Finding 6: `mondayMetadataSchema.userIdentity` uses an anonymous inline schema

- **File**: `packages/contracts/src/contracts/integrations/platforms/monday/platform-metadata.schema.ts:14`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: Same issue as Notion — `mondayMetadataSchema` defines `userIdentity` as an anonymous inline `z.object({ email: z.string(), name: z.string(), userId: z.string() }).optional()` without `.strict()` and without being exported as a named schema. The `email` field here also uses `z.string()` instead of `z.email()`.
- **Suggestion**: Extract `mondayUserIdentitySchema = z.object({ email: z.email(), name: z.string(), userId: z.string() }).strict()` as a named export.
- **Evidence**: Inline `z.object({ email: z.string(), name: z.string(), userId: z.string() }).optional()` at line 14.
