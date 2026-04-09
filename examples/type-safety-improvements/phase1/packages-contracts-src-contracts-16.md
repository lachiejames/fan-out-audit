# Audit: packages-contracts-src-contracts-16

**Files inspected**: 8
**Findings**: 6

## Summary

This slice covers suggestions types, sync-progress schemas, templates types, test-fixture schemas, todo-extraction types, todos constants and types. The sync-progress file is the largest in this slice and is well-structured. The test-fixtures file is extensive and covers E2E-only operations. The main concerns are: `integrationPlatformSchema` is re-declared in `sync-progress/schemas.ts` rather than imported from the canonical location, `createMessageBodySchema.platformIdentifiers` uses a double-unknown record, multiple test-fixture schemas have `Record<string, unknown>` for metadata/source fields, and the `knowledgeFactSchema.category` in sync-progress duplicates the enum from `knowledge/schemas.ts`.

## Findings

### Finding 1: `integrationPlatformSchema` is re-declared in `sync-progress/schemas.ts`

- **File**: `packages/contracts/src/contracts/sync-progress/schemas.ts:24`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `export const integrationPlatformSchema = z.string().min(1)` is declared at line 24 in `sync-progress/schemas.ts`. The canonical `integrationPlatformSchema` is defined in `contracts/integrations/schemas.ts`. This creates two independent schemas with the same name and same definition. The `syncProgressEventResponseSchema.platform` field uses the local copy at line 340, while `platformSyncStatusSchema.platform` also uses the local copy. If the canonical schema ever gains validation constraints (e.g., an enum), the sync-progress copy would silently drift.
- **Suggestion**: Remove the re-declaration from `sync-progress/schemas.ts` and import `integrationPlatformSchema` from `@/contracts/integrations/schemas.ts`.
- **Evidence**: `export const integrationPlatformSchema = z.string().min(1);` at line 24, duplicating the definition in `integrations/schemas.ts`.

### Finding 2: `knowledgeFactSchema.category` duplicates the knowledge category enum

- **File**: `packages/contracts/src/contracts/sync-progress/schemas.ts:255`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `knowledgeFactSchema` at line 255 defines `category: z.enum(["context", "fact", "preference", "relationship", "skill", "style"])` inline. This is an identical copy of `knowledgeCategorySchema` from `contracts/knowledge/schemas.ts`. The two can drift independently if a new category is added to one but not the other.
- **Suggestion**: Import `knowledgeCategorySchema` from `@/contracts/knowledge/schemas.ts` and use it in place of the inline enum.
- **Evidence**: `category: z.enum(["context", "fact", "preference", "relationship", "skill", "style"]),` at line 257 in `sync-progress/schemas.ts`.

### Finding 3: `createMessageBodySchema.platformIdentifiers` uses `z.record(z.string(), z.unknown())`

- **File**: `packages/contracts/src/contracts/test-fixtures/schemas.ts:192`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `platformIdentifiers: z.record(z.string(), z.unknown()).nullable().optional()` in the test-fixture `createMessageBodySchema` accepts any object as platform identifiers. The `actionTargetIdentifiersSchema` from `core/platform-identifiers.schemas` already provides a typed union for platform identifiers. Using a generic record here weakens the E2E test fixture validation.
- **Suggestion**: Replace `z.record(z.string(), z.unknown())` with the existing `actionTargetIdentifiersSchema` from `@/contracts/integrations/core/platform-identifiers.schemas`. The import is already used elsewhere in this file (`createWorkItemBodySchema` at line 248).
- **Evidence**: `platformIdentifiers: z.record(z.string(), z.unknown()).nullable().optional(),` at line 192.

### Finding 4: `createTodoBodySchema.source` in test-fixtures uses `z.record(z.string(), z.unknown())`

- **File**: `packages/contracts/src/contracts/test-fixtures/schemas.ts:343`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `source: z.record(z.string(), z.unknown()).nullable().optional()` on the test-fixture `createTodoBodySchema` is a free-form object. The production `createTodoBodySchema` (in `todos/schemas.ts`) stores source context as a typed object. Using a generic record in the fixture schema means E2E tests could pass malformed source data without validation catching it.
- **Suggestion**: Align the test-fixture `createTodoBodySchema.source` field type with the production todos schema's source type, or at minimum use `z.record(z.string(), z.union([z.string(), z.number(), z.boolean(), z.null()]))` to prevent deeply nested unknown objects.
- **Evidence**: `source: z.record(z.string(), z.unknown()).nullable().optional(),` at line 343.

### Finding 5: `createIntegrationBodySchema.metadata` in test-fixtures uses `z.record(z.string(), z.unknown())`

- **File**: `packages/contracts/src/contracts/test-fixtures/schemas.ts:434`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `metadata: z.record(z.string(), z.unknown())` on the fixture `createIntegrationBodySchema` accepts any object as integration platform metadata. The production `platformMetadataSchema` provides per-platform typed schemas. The fixture version intentionally omits this validation to simplify test setup, but it silently allows malformed metadata that would fail in production.
- **Suggestion**: Add a comment documenting why `Record<string, unknown>` is used in the fixture (simplified test setup), so future maintainers understand the trade-off was intentional.
- **Evidence**: `metadata: z.record(z.string(), z.unknown()),` at line 434.

### Finding 6: `syncStreamProgressEventSchema` and `syncProgressEventResponseSchema` are nearly identical — no shared base

- **File**: `packages/contracts/src/contracts/sync-progress/schemas.ts:216` and `316`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `syncStreamProgressEventSchema` (SSE format, lines 216-249) and `syncProgressEventResponseSchema` (REST format, lines 316-348) share nearly all fields: `createdAt`, `currentItem`, `error`, `etaSeconds`, `id`, `messagesIndexed`, `messagesSkipped`, `messagesSynced`, `metrics`, `percent`, `phase`, `platform`, `processedItems`, `syncRunId`, `threadsIndexed`, `timestamp`, `tokensUsed`, `total`. The only structural differences are the `type: z.literal("progress")` field in the SSE version and `userId` in the SSE version.
- **Suggestion**: Extract a `syncProgressEventBaseSchema` containing the shared fields, then derive `syncStreamProgressEventSchema` by adding `type` and `syncProgressEventResponseSchema` by adding `userId`.
- **Evidence**: Both schemas define the same `messagesIndexed`, `messagesSynced`, `messagesSkipped`, `threadsIndexed`, `tokensUsed`, `metrics`, `currentItem` etc. — approximately 15 shared fields.
