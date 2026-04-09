# Audit: packages-contracts-src-contracts-18

**Files inspected**: 4
**Findings**: 5

## Summary

This slice covers voice vocabulary, work-items schemas, work-items types, and workflows types. The work-items file is the most significant in the entire audit — it is the largest schema file in the contracts package and defines a large discriminated union of action types. The main concerns are: `aiSources: z.array(z.record(z.string(), z.unknown()))` on `baseWorkItemSchema`, `columnValues: z.record(z.string(), z.unknown())` on Monday action schemas, `properties: z.record(z.string(), z.unknown())` on Notion action schemas, and `z.string()` timestamp fields throughout the base schema.

## Findings

### Finding 1: `baseWorkItemSchema.aiSources` uses `z.array(z.record(z.string(), z.unknown()))`

- **File**: `packages/contracts/src/contracts/work-items/schemas.ts:131`
- **Category**: any-usage
- **Impact**: medium
- **Description**: `aiSources: z.array(z.record(z.string(), z.unknown())).nullable()` stores the AI source references used to generate a work item. These are likely content IDs or content references with known shapes (e.g., `{ contentId: uuid, source: string, snippet: string }`). Using `z.array(z.record(z.string(), z.unknown()))` means any object array passes validation, making the field opaque to both the frontend and backend.
- **Suggestion**: Define an `aiSourceSchema = z.object({ contentId: z.uuid().optional(), source: z.string().optional(), snippet: z.string().optional() })` and use `z.array(aiSourceSchema).nullable()`. The current loose type is likely a historical artefact from before the AI source shape was stabilized.
- **Evidence**: `aiSources: z.array(z.record(z.string(), z.unknown())).nullable(),` at line 131.

### Finding 2: `baseWorkItemSchema` timestamp fields use `z.string()` instead of `isoDateTimeStringSchema`

- **File**: `packages/contracts/src/contracts/work-items/schemas.ts:138-172`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: Multiple timestamp fields in `baseWorkItemSchema` use raw `z.string()`: `approvedAt`, `createdAt`, `executedAt`, `expiresAt`, `scheduledFor`, `undoDeadline`, `updatedAt`. These should use `isoDateTimeStringSchema` per the repo-wide convention established in `schemas.ts`. This is the most systematic violation of the timestamp convention in the work-items contract.
- **Suggestion**: Replace all `z.string()` timestamp fields in `baseWorkItemSchema` with `isoDateTimeStringSchema` (imported from `@/schemas.ts`). The nullable variants should use `isoDateTimeStringSchema.nullable()`.
- **Evidence**: `approvedAt: z.string().nullable(),` at line 133, `createdAt: z.string(),` at line 139, `expiresAt: z.string(),` at line 144, etc.

### Finding 3: `columnValues: z.record(z.string(), z.unknown())` on Monday action schemas

- **File**: `packages/contracts/src/contracts/work-items/schemas.ts:451` and `463`
- **Category**: record-weakening
- **Impact**: low
- **Description**: Both `createMondayItemActionSchema.metadata.columnValues` and `updateMondayItemActionSchema.metadata.columnValues` use `z.record(z.string(), z.unknown())`. Monday.com column values can be strings, numbers, or structured objects depending on the column type. Since column IDs are board-specific and cannot be statically enumerated, `Record<string, unknown>` is the appropriate type for the key. However the value type could be narrowed to `z.union([z.string(), z.number(), z.boolean(), z.null(), z.object({}).passthrough()])`.
- **Suggestion**: At minimum, change to `z.record(z.string(), z.union([z.string(), z.number(), z.boolean(), z.null()]))` to prevent deeply nested unknown structures that would fail Monday.com's API.
- **Evidence**: `columnValues: z.record(z.string(), z.unknown()).optional(),` at line 451.

### Finding 4: `properties: z.record(z.string(), z.unknown())` on Notion action schemas

- **File**: `packages/contracts/src/contracts/work-items/schemas.ts:491` and `507`
- **Category**: record-weakening
- **Impact**: low
- **Description**: Both `createNotionPageActionSchema.metadata.properties` and `updateNotionPageActionSchema.metadata.properties` use `z.record(z.string(), z.unknown())`. Notion database properties require structured values (e.g., `{ type: "title", title: [{ text: { content: "..." } }] }`). The `NotionPropertyValue` interface is already defined in the Notion expanded-data schema and could be reused here.
- **Suggestion**: Import `notionPropertyValueSchema` from `contracts/integrations/platforms/notion/expanded-data.schema.ts` and replace `z.record(z.string(), z.unknown())` with `z.record(z.string(), notionPropertyValueSchema)` in both Notion action schemas.
- **Evidence**: `properties: z.record(z.string(), z.unknown()).optional(),` at line 491.

### Finding 5: `deleteVocabularyItem` in `voice/vocabulary.ts` uses `z.object({})` as body — should be `emptyBodySchema`

- **File**: `packages/contracts/src/contracts/voice/vocabulary.ts:107`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `deleteVocabularyItem` uses `body: z.object({})` for its empty body. The repo convention (documented in `CLAUDE.md` and used throughout other contracts) is to use `emptyBodySchema` from `@/schemas.ts` for mutations with no body. Using a raw `z.object({})` skips the `.strict()` enforcement that `emptyBodySchema` provides via its `z.object({}).strict().optional().default({})` definition.
- **Suggestion**: Import `emptyBodySchema` from `@/schemas.ts` and replace `z.object({})` with `emptyBodySchema`.
- **Evidence**: `body: z.object({}),` at line 107 in `voice/vocabulary.ts`.
