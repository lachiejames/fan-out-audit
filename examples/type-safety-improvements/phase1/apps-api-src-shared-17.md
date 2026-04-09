# Audit: apps-api-src-shared-17

**Files inspected**: 8
**Findings**: 2

## Summary

The four platform error extractor utilities in this slice (`github-api-error.utils.ts`, `google-api-error.utils.ts`, `http-fetch-error.utils.ts`, `linear-error.utils.ts`) all share the same structural `as Record<string, unknown>` cast pattern for accessing duck-typed properties on caught unknown errors. One file (`media-validation.utils.ts`) contains a commented-out type that was never implemented.

## Findings

### Finding 1: Repeated `as Record<string, unknown>` cast pattern across all four platform error extractors

- **File**: `apps/api/src/shared/utils/github-api-error.utils.ts`, `google-api-error.utils.ts`, `http-fetch-error.utils.ts`, `linear-error.utils.ts`
- **Category**: type-cast
- **Impact**: low
- **Description**: All four files follow the same pattern: catch `unknown`, check `typeof value === "object" && value !== null`, then cast to `Record<string, unknown>` to access platform-specific error fields (e.g. `status`, `code`, `message`, `errors[]`). Each file duplicates the same guard+cast boilerplate. While individually safe, the repetition increases maintenance burden and the casts are still technically unsafe.
- **Suggestion**: Extract the shared `isRecord(value: unknown): value is Record<string, unknown>` guard into `shared/utils/type-guards.utils.ts` (or reuse the `isObjectLike` guard already present in `postgres-error.utils.ts`). Each platform extractor can then call `if (isRecord(error))` and access properties without the `as` cast. This eliminates four identical cast sites in a single change.
- **Evidence**:
  ```ts
  // Repeated across all four files:
  if (typeof error === "object" && error !== null) {
    const errorRecord = error as Record<string, unknown>;
    // access errorRecord.status, errorRecord.code, etc.
  }
  ```

### Finding 2: Commented-out `AllowedImageType` type in `media-validation.utils.ts`

- **File**: `apps/api/src/shared/utils/media-validation.utils.ts:25-26`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: Lines 25–26 contain a commented-out `AllowedImageType` type definition that was never implemented. The active code uses a plain `string[]` constant `ALLOWED_IMAGE_TYPES` and compares against it at runtime, providing no compile-time enforcement of valid image MIME types.
- **Suggestion**: Uncomment and activate `AllowedImageType` as a string literal union (e.g. `"image/jpeg" | "image/png" | "image/webp" | "image/gif"`), derive `ALLOWED_IMAGE_TYPES` from it with `satisfies AllowedImageType[]`, and use `AllowedImageType` as the return type for the validation pass case. This gives callers compile-time type narrowing on the accepted MIME type string.
- **Evidence**:
  ```ts
  // type AllowedImageType = "image/jpeg" | "image/png" | ...  (commented out)
  const ALLOWED_IMAGE_TYPES = ["image/jpeg", "image/png", ...]; // string[]
  ```

## No additional findings

`indexing-budget.utils.ts`, `invariant.ts`, and `parse-json.ts` are clean: they use `unknown`-based narrowing, explicit return types, and no unsafe casts.
