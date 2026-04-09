# Audit: packages-contracts-src-schemas.ts

**Files inspected**: 1
**Findings**: 3

## Summary

`schemas.ts` is the shared infrastructure layer for the contracts package — it provides `isoDateTimeStringSchema`, `emptyObjectSchema`/`emptyBodySchema`/`emptyQuerySchema`, `queryBooleanSchema`, `errorResponseSchema`, `standardResponses()`, `paginatedResponse()`, and the `contentDtoSchema`/`integrationMessageDtoSchema`/`chatMessageDtoSchema` DTOs. The file is well-structured. The main concerns are: the `contentDtoSchema.metadata` field uses a raw record, and there is a TODO comment noting that the DTO schemas should be moved to local contract files.

## Findings

### Finding 1: `contentDtoSchema.metadata` uses `z.record(z.string(), z.unknown())`

- **File**: `packages/contracts/src/schemas.ts:228`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: `contentDtoSchema.metadata: z.record(z.string(), z.unknown())` is the base metadata field for all content objects. This field is used across messages, search, and content contracts. Since the CLAUDE.md explicitly calls out `contracts/integrations/registry.ts` as the authority for per-platform metadata schemas, and `decodeContentMetadataStrict` already provides typed decoding, the raw `Record<string, unknown>` type on `contentDtoSchema` means that anywhere the content DTO is used directly, metadata access is untyped.
- **Suggestion**: This is a systemic issue. The TODO comment at line 205 (`// TODO: Move these to local contract schemas once refactored`) acknowledges the technical debt. As a stepping stone, add JSDoc to `contentDtoSchema.metadata` referencing the `decodeContentMetadataStrict` function as the proper way to access typed metadata.
- **Evidence**: `metadata: z.record(z.string(), z.unknown()),` at line 228.

### Finding 2: `errorResponseSchema.details` uses `z.record(z.string(), z.unknown())`

- **File**: `packages/contracts/src/schemas.ts:133`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `details: z.record(z.string(), z.unknown()).optional()` on `errorResponseSchema` is intentionally open-ended to allow any structured error details from backend validation. This is an acceptable use of `Record<string, unknown>` since error detail shapes vary by endpoint. However, the field type could be `z.record(z.string(), z.union([z.string(), z.number(), z.array(z.string())]))` to prevent deeply nested unknown objects in error details.
- **Suggestion**: Consider narrowing to `z.record(z.string(), z.union([z.string(), z.number(), z.boolean(), z.null(), z.array(z.string())]))` since error detail values are typically flat primitives or string arrays.
- **Evidence**: `details: z.record(z.string(), z.unknown()).optional(),` at line 133.

### Finding 3: `paginatedResponse` factory function has no explicit return type annotation

- **File**: `packages/contracts/src/schemas.ts:195`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `export function paginatedResponse<T extends z.ZodType>({ itemSchema }: { itemSchema: T })` has no explicit return type. Per the `typescript-patterns.md` rule, all exported functions must have explicit return types. The TypeScript compiler can infer the return type, but explicitly annotating it would improve IDE experience and prevent accidental structural changes.
- **Suggestion**: Add an explicit return type: `function paginatedResponse<T extends z.ZodType>({ itemSchema }: { itemSchema: T }): z.ZodObject<{ hasMore: z.ZodBoolean; items: z.ZodArray<T>; totalCount: z.ZodNumber }>`.
- **Evidence**: `export function paginatedResponse<T extends z.ZodType>({ itemSchema }: { itemSchema: T }) {` at line 195 — no return type annotation.
