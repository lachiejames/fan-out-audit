# Audit: apps-api-src-shared-16

**Files inspected**: 8
**Findings**: 3

## Summary

Most utilities in this slice are well-typed with proper `unknown`-based error handling. The main findings are: `ActionPreview.platform` typed as `string` instead of the narrower `PlatformId` contract type, multiple `as Record<string, unknown>` casts in `ai-sdk-error.utils.ts` for duck-typed error object access, and `MIME_TYPE_EXTRACTORS` in `document-text-extractor.utils.ts` keyed by `string` when the valid keys form a closed set.

## Findings

### Finding 1: `ActionPreview.platform` typed as `string` instead of `PlatformId`

- **File**: `apps/api/src/shared/types/work-items.types.ts:19`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `ActionPreview.platform` is typed as `string`. The `@slopweaver/contracts` package already exports `PlatformId` (a generated discriminated union of all platform IDs). Using `string` allows any platform name to be stored without a compile-time check.
- **Suggestion**: Replace `platform: string` with `platform: PlatformId` imported from `@slopweaver/contracts`. If `ActionPreview` can represent a hypothetical/future platform not yet registered, use `PlatformId | (string & {})` with a comment explaining why.
- **Evidence**:
  ```ts
  export type ActionPreview = {
    platform: string;
    // ...
  };
  ```

### Finding 2: `as Record<string, unknown>` casts in `ai-sdk-error.utils.ts`

- **File**: `apps/api/src/shared/utils/ai-sdk-error.utils.ts:38,53,68,70`
- **Category**: type-cast
- **Impact**: low
- **Description**: The error extraction helpers cast caught `unknown` values to `Record<string, unknown>` in order to access duck-typed properties (e.g. `error.statusCode`, `error.responseBody`). While the surrounding `typeof` guards narrow `unknown` to `object`, the subsequent `as Record<string, unknown>` casts bypass TypeScript's structural checking. If the AI SDK exports typed error classes, those should be used directly; otherwise a typed type guard would be safer than a cast.
- **Suggestion**: Replace `as Record<string, unknown>` with a proper type guard function such as `isRecord(value: unknown): value is Record<string, unknown>` that checks `typeof value === "object" && value !== null`. This is already available in `postgres-error.utils.ts` as `isObjectLike` — extract it to a shared utility and reuse it here.
- **Evidence**:
  ```ts
  const errorRecord = error as Record<string, unknown>;
  const statusCode = errorRecord["statusCode"];
  ```

### Finding 3: `MIME_TYPE_EXTRACTORS` keyed by `string` in `document-text-extractor.utils.ts`

- **File**: `apps/api/src/shared/utils/document-text-extractor.utils.ts:102`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `MIME_TYPE_EXTRACTORS` is typed as `Record<string, ExtractorFn>`. The map is initialised with a fixed set of MIME type keys (e.g. `"application/pdf"`, `"text/plain"`). Using `string` as the key type means any string lookup compiles without error, masking typos in MIME strings at call sites.
- **Suggestion**: Extract the known MIME types to a union (e.g. `type SupportedMimeType = "application/pdf" | "text/plain" | ...`) and type the map as `Record<SupportedMimeType, ExtractorFn>`. This makes unsupported lookups a compile-time error rather than a silent `undefined`.
- **Evidence**:
  ```ts
  const MIME_TYPE_EXTRACTORS: Record<string, ExtractorFn> = {
    "application/pdf": extractPdfText,
    "text/plain": extractPlainText,
    // ...
  };
  ```

## No additional findings

`content-sanitization.utils.ts`, `data-url.utils.ts`, `datetime.ts`, `elevenlabs-error.utils.ts`, and `embedding-text.utils.ts` are otherwise clean: they use `unknown` + type guards for error handling and have no unsafe casts or `any` usage.
