# Audit: packages-contracts-src-contracts-13

**Files inspected**: 8
**Findings**: 7

## Summary

This slice covers the integration registry, integration schemas, knowledge-sources schemas, knowledge schemas, knowledge types, and memory export. The registry is well-designed with strongly-typed platform constants and helper functions. The main issues are: `CONTENT_METADATA_SCHEMAS_BY_TYPE_AND_SOURCE` using `Partial<Record<ContentType, Partial<Record<string, z.ZodType>>>>` where a stricter inner key could be used, several `z.record(z.string(), z.unknown())` usages in knowledge-source schemas where more specific shapes are known, and `KnowledgeStatus` type duplication between `knowledge/schemas.ts` and `knowledge/types.ts`.

## Findings

### Finding 1: `KnowledgeStatus` type is exported from both `schemas.ts` and `types.ts`

- **File**: `packages/contracts/src/contracts/knowledge/schemas.ts:11` and `packages/contracts/src/contracts/knowledge/types.ts:4`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `KnowledgeStatus` is exported as `export type KnowledgeStatus = z.infer<typeof knowledgeStatusSchema>` in both `knowledge/schemas.ts` (line 11) and `knowledge/types.ts` (line 4). Consumers that import from either path get an identical type but with different import paths. If the schema changes, both files must be updated. The same pattern exists for `KnowledgeCategory` but it is only in `types.ts`.
- **Suggestion**: Remove the `KnowledgeStatus` type export from `knowledge/schemas.ts` and ensure all imports go through `knowledge/types.ts`, which is the designated type aggregation file.
- **Evidence**:
  ```ts
  // knowledge/schemas.ts line 11
  export type KnowledgeStatus = z.infer<typeof knowledgeStatusSchema>;
  // knowledge/types.ts line 4
  export type KnowledgeStatus = z.infer<typeof knowledgeStatusSchema>;
  ```

### Finding 2: `CONTENT_METADATA_SCHEMAS_BY_TYPE_AND_SOURCE` inner `Record<string, z.ZodType>` is untyped

- **File**: `packages/contracts/src/contracts/integrations/registry.ts:117`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: `CONTENT_METADATA_SCHEMAS_BY_TYPE_AND_SOURCE` is typed as `Partial<Record<ContentType, Partial<Record<string, z.ZodType>>>>`. The inner `Record<string, z.ZodType>` maps platform IDs to their metadata schemas, but platform IDs are untyped strings. This means typos in platform IDs (e.g., `"google_gmail"` vs `"google-gmail"`) would compile without error.
- **Suggestion**: Change the inner record type from `Partial<Record<string, z.ZodType>>` to `Partial<Record<PlatformId, z.ZodType>>`. The `PlatformId` union type is already available in this file.
- **Evidence**: `Partial<Record<ContentType, Partial<Record<string, z.ZodType>>>>` at line 117.

### Finding 3: `unknownObjectSchema` fallback in `decodeMessageMetadataStrict` loses type information

- **File**: `packages/contracts/src/contracts/integrations/registry.ts:109` and `289`
- **Category**: any-usage
- **Impact**: low
- **Description**: `const unknownObjectSchema = z.record(z.string(), z.unknown())` is used as a fallback schema for unrecognized platforms. The overload signature correctly types the unknown-platform path as `Record<string, unknown>`. This is intentionally loose and documented, but the fallback silently accepts any object structure from an unrecognized platform. A runtime warning log when the fallback is hit would help diagnose configuration issues.
- **Suggestion**: No type change needed (the design is intentional), but consider adding a `console.warn` or a `logger.warn` call in the `decodeMessageMetadataStrict` fallback branch for unknown platforms.
- **Evidence**: `const unknownObjectSchema = z.record(z.string(), z.unknown());` at line 109.

### Finding 4: `knowledgeSourceRevisionResponseSchema.captureMetadata` uses `z.record(z.string(), z.unknown())`

- **File**: `packages/contracts/src/contracts/knowledge-sources/schemas.ts:313`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `captureMetadata: z.record(z.string(), z.unknown()).nullable()` stores platform-specific capture configuration. For connected snapshot sources (e.g., Drive folders), this would contain known fields like `folderId`, `folderName`, `includeSubfolders`. Using a generic record means any component reading `captureMetadata` must perform unsafe runtime casting.
- **Suggestion**: Define a discriminated union type for capture metadata (e.g., `DriveCaptureMetadata`, `ManualCaptureMetadata`) keyed by the source's `kind` or `provider` field, and use that union instead of `Record<string, unknown>`.
- **Evidence**: `captureMetadata: z.record(z.string(), z.unknown()).nullable(),` at line 313.

### Finding 5: `selectedScopeSchema.params` uses `z.record(z.string(), z.unknown())`

- **File**: `packages/contracts/src/contracts/knowledge-sources/schemas.ts:284`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `selectedScopeSchema` has `params: z.record(z.string(), z.unknown()).optional()`. The `params` field holds configuration for the selected scope option (e.g., folder depth, page limit for a website snapshot). Since scope options are defined by `SCOPE_OPTION_IDS` in the backend, the possible params for each option could be statically typed.
- **Suggestion**: Define named param schemas for each scope option ID and use a discriminated union (or at minimum `Record<string, string | number | boolean>` instead of `Record<string, unknown>`).
- **Evidence**: `params: z.record(z.string(), z.unknown()).optional(),` at line 284.

### Finding 6: `knowledgeSourceItemResponseSchema.metadata` uses `z.record(z.string(), z.unknown())`

- **File**: `packages/contracts/src/contracts/knowledge-sources/schemas.ts:368`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `metadata: z.record(z.string(), z.unknown()).nullable()` on `knowledgeSourceItemResponseSchema`. This is the item-level metadata for knowledge source items (conversations, project documents, etc.). Like `captureMetadata`, the shape is provider-specific and could benefit from a discriminated type.
- **Suggestion**: As a minimum improvement, change to `z.record(z.string(), z.union([z.string(), z.number(), z.boolean(), z.null()])).nullable()` since metadata values are unlikely to be deeply nested objects. A discriminated union per `itemType` would be the ideal solution.
- **Evidence**: `metadata: z.record(z.string(), z.unknown()).nullable(),` at line 368.

### Finding 7: `knowledgeSourceImportSummarySchema.categoryCounts` uses `z.record(z.string(), z.number())`

- **File**: `packages/contracts/src/contracts/knowledge-sources/schemas.ts:231`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `categoryCounts: z.record(z.string(), z.number().int().min(0))` counts knowledge items per category. The valid category keys are defined by `knowledgeCategorySchema` (`"preference"`, `"fact"`, `"relationship"`, `"skill"`, `"style"`, `"context"`). Using `z.string()` as the key allows any string key including typos.
- **Suggestion**: Change the key schema to use the enum: `z.record(knowledgeCategorySchema, z.number().int().min(0))`. Zod v4 supports enum keys in `z.record()`.
- **Evidence**: `categoryCounts: z.record(z.string(), z.number().int().min(0)),` at line 231.
