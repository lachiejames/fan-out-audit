# Audit: packages-contracts-src-contracts-6

**Files inspected**: 8
**Findings**: 8

## Summary

This slice covers email and calendar expanded-data schemas, file storage schemas, generated union files, indexing budget schemas, and platform identifier schemas. The generated files (`expanded-data.ts`, `platform-metadata.ts`, `platforms.ts`) are well-structured. The main issues are: `storageFileSchema.platformMetadata` is opaque, several `email-expanded-data` fields lack format validation, `UnifiedCalendarEvent` duplicates interface/schema, and `externalIdPlatformIdentifiersSchema` constructs a union dynamically with `.map()` which has a known TypeScript type-inference limitation.

## Findings

### Finding 1: `storageFileSchema.platformMetadata` and `storageFolderSchema.platformMetadata` are `z.record(z.string(), z.unknown())`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/core/file-storage-schemas.ts:92` and `128`
- **Category**: any-usage
- **Impact**: medium
- **Description**: Both `StorageFile.platformMetadata` and `StorageFolder.platformMetadata` are fully opaque records. Platform-specific metadata for files (e.g., Google Drive `driveId`, OneDrive `eTag`) is untyped.
- **Suggestion**: Define per-platform file metadata interfaces similar to how content metadata is handled. At minimum, document the known keys per platform in JSDoc.
- **Evidence**: `platformMetadata: z.record(z.string(), z.unknown()),` at line 92 of `file-storage-schemas.ts`.

### Finding 2: `emailAddressSchema.email` is `z.string()` — not validated as email format

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/core/email-expanded-data.schema.ts:58`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `emailAddressSchema.email` is `z.string()` without email format validation, even though the field is explicitly named `email`. The type `EmailAddress` has `email: string` — at minimum this should be `z.string().email()` or the Zod 4 equivalent `z.email()`.
- **Suggestion**: Change to `z.email()` (Zod 4 built-in). This is used in email thread rendering and will catch malformed email strings from the API.
- **Evidence**: `email: z.string(),` at line 59 of the `emailAddressSchema`.

### Finding 3: `emailMessageSchema.timestamp` is `z.string()` without ISO datetime validation

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/core/email-expanded-data.schema.ts:82`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `timestamp: z.string()` on email messages. The interface comment says "ISO string" but there's no format validation. Other parts of the codebase use `isoDateTimeStringSchema`.
- **Suggestion**: Use `isoDateTimeStringSchema` for consistency.
- **Evidence**: `timestamp: z.string(),` at line 82; comment `// ISO string` on `EmailMessage.timestamp`.

### Finding 4: `UnifiedCalendarEvent` interface duplicates the Zod schema shape — maintenance burden

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/core/calendar-expanded-data.schema.ts:9-50`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `UnifiedCalendarEvent` and `UnifiedCalendarAttendee` are defined as explicit TypeScript interfaces, then bound to `z.ZodType<UnifiedCalendarEvent>`. This requires maintaining both in sync. If a new field is added to the Zod schema but not the interface (or vice versa), TypeScript will catch it — but the boilerplate is higher than necessary.
- **Suggestion**: For schemas of this complexity, `z.infer<typeof unifiedCalendarEventSchema>` would avoid the parallel interface. Only use the explicit interface pattern when TS7056 is a real concern (schemas in large discriminated unions with `.passthrough()`). These schemas are relatively flat and unlikely to trigger TS7056.
- **Evidence**: Interface `UnifiedCalendarEvent` at lines 15-26, schema at lines 34-50.

### Finding 5: `externalIdPlatformIdentifiersSchema` uses `z.union(array.map(...))` — TypeScript loses the element type

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/core/platform-identifiers.schemas.ts:73-75`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `z.union(externalIdPlatformIds.map(p => makeExternalIdIdentifiersSchema({ platform: p })))` constructs a union from a mapped array. TypeScript types this as `ZodUnion<ZodObject<...>[]>` not `ZodUnion<[ZodObject<...>, ZodObject<...>, ...]>` (tuple), which means `z.infer` produces a union of the general shape rather than the specific per-platform discriminated shapes. The type guard `isExternalIdPlatformIdentifiers` works at runtime, but static narrowing by `platform` is lost.
- **Suggestion**: This is a known TypeScript limitation with dynamically constructed unions. The current pattern is acceptable given the number of platforms. Add a comment explaining why a static union cannot be used. If strong narrowing per `platform` is needed, consider generating explicit union members in the codegen step.
- **Evidence**: `export const externalIdPlatformIdentifiersSchema = z.union(externalIdPlatformIds.map((p) => makeExternalIdIdentifiersSchema({ platform: p })));` at lines 73-75.

### Finding 6: `budgetStatusResponseSchema.platformBreakdown` key type is `integrationPlatformSchema` (string) in a `z.record`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/core/indexing-budget.schemas.ts:9`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `z.record(integrationPlatformSchema, z.number().int().min(0))` — `integrationPlatformSchema` is `z.string()`, so the record key type is `string`. Zod doesn't use the key schema for indexing in TypeScript types (always produces `Record<string, V>`), but the intent to constrain to known platform IDs is undermined.
- **Suggestion**: This is a Zod limitation — `z.record(keySchema, valueSchema)` always types the key as `string` in TypeScript regardless of `keySchema`. Document this limitation with a comment, and consider post-processing the record to validate keys at the service layer.
- **Evidence**: `platformBreakdown: z.record(integrationPlatformSchema, z.number().int().min(0)).nullable(),` at line 9.

### Finding 7: `fileStorageMetadataSchema.webViewLink` uses `z.url()` but `storageFileSchema.downloadLink` uses nullable `z.url()` — inconsistency

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/core/file-storage-schemas.ts:346` and `69`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `storageFileSchema.downloadLink` is `z.url().nullable()` and `storageFileSchema.webViewLink` is `z.url()` (required). But `fileStorageMetadataSchema.webViewLink` is also `z.url()` (required). However `storageFolderItemSchema` has `webViewLink: z.url()` required too. The inconsistency is that `downloadLink` can be null (file has no download link) but `webViewLink` cannot — however some platforms return null web view links. Review if this should be nullable.
- **Suggestion**: Audit whether `webViewLink` can realistically be null from any platform. If so, change to `z.url().nullable()` with appropriate handling in the frontend.
- **Evidence**: `webViewLink: z.url(),` at lines 96 and 129 vs `downloadLink: z.url().nullable(),` at line 69.

### Finding 8: `PassthroughMetadata<T>` type utility creates an index signature conflict

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/platform-metadata-shared.ts:11`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `export type PassthroughMetadata<T> = T & Record<string, unknown>` — intersecting with `Record<string, unknown>` allows any string key on the type, which means field-level access control (e.g., accessing a non-existent field) returns `unknown` rather than a type error. This is intentional for `.loose()` schema support, but it means TypeScript cannot flag misspelled field access on `AsanaMetadata`, `GithubMetadata`, etc.
- **Suggestion**: This is a known trade-off. An alternative is to add a `[key: string]: unknown` index signature directly on each metadata interface (as done in `asana/expanded-data.schema.ts` for `AsanaUser`) rather than via an intersection, which can result in slightly cleaner TypeScript error messages.
- **Evidence**: `export type PassthroughMetadata<T> = T & Record<string, unknown>;` at line 11.
