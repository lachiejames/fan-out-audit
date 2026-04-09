# Audit: apps-api-src-application-22

**Files inspected**: 8
**Findings**: 4

## Summary

These files cover knowledge CRUD/search, LLM call audit logging, media file storage/management, and domain error types for media files and memory. The code is generally well-typed and follows the project's Result/neverthrow patterns. Four type-safety issues were found: a loose union type on `AuditSuspiciousActivity.severity` that defeats exhaustive handling, untyped `storageBucket` and `mediaType` parameters in `MediaFilesStorageService` that accept arbitrary strings instead of constrained literals, and a duplicate `MediaFileDTO` local type alias derived three separate times across files.

## Findings

### Finding 1: `AuditSuspiciousActivity.severity` union is intentionally widened with `| string`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/types/ai.types.ts:11`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: The `severity` field is typed as `"low" | "medium" | "high" | string`. The trailing `| string` collapses the discriminated union to plain `string`, meaning TypeScript will not narrow it and switch exhaustiveness checks against the literal union are impossible. In `llm-calls.service.ts:176` a guard `a.severity === "high"` still compiles, but the type offers no safety — any string silently passes. This is consumed directly from the AI response and deserves a strict union.
- **Suggestion**: Remove `| string`. If unknown values can arrive from the AI, handle them with a type guard that validates against the known literals before constructing the object, or add a catch-all `"unknown"` literal instead of a bare `string`.
- **Evidence**:

```typescript
// apps/api/src/shared/types/ai.types.ts
export type AuditSuspiciousActivity = {
  details: string;
  severity: "low" | "medium" | "high" | string; // | string defeats the union
  type: string;
};
```

### Finding 2: `storageBucket` parameter typed as `string` instead of a constrained literal union

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/application/media-files/services/media-files-storage.service.ts:57`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `uploadFile`, `uploadBuffer`, `downloadFileBuffer`, `deleteStorageFile`, and `createSignedUrl` all accept `storageBucket?: string` (default `"user-uploads"`). The codebase uses exactly two bucket names: `"user-uploads"` and `"chat-images"`. Typing this as plain `string` means a typo at any call-site compiles silently and causes a runtime storage error. A literal union would surface misuses at compile time.
- **Suggestion**: Define `type StorageBucket = "user-uploads" | "chat-images"` and use it as the parameter type for all `storageBucket` parameters in this file. The default value `"user-uploads"` already satisfies the narrower type.
- **Evidence**:

```typescript
// media-files-storage.service.ts — repeated across 5 methods
async uploadFile({
  storageBucket = "user-uploads",
  ...
}: {
  storageBucket?: string;   // accepts any string
  ...
})

// The only two values ever used in this file:
const storageBucket = isImage ? "chat-images" : "user-uploads";
```

### Finding 3: Duplicate `MediaFileDTO` local type derived three times

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/application/media-files/services/media-files-storage.service.ts:18-19`, `/Users/lachiejames/dev/slopweaver/apps/api/src/application/media-files/services/media-files.service.ts:15-16`, `/Users/lachiejames/dev/slopweaver/apps/api/src/application/media-files/services/media-files.utils.ts:14-15`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: All three files independently derive a local `MediaFileDTO` type alias using the same two-line pattern. If the contract shape changes, or the extraction logic needs updating, all three sites must be changed in sync. The utility file `media-files.utils.ts` is already the canonical mapping location and exports `toMediaFileDto`; it is the natural home for this shared type alias.
- **Suggestion**: Export `MediaFileDTO` once from `media-files.utils.ts` (or a dedicated `media-files.types.ts` file) and import it in the other two service files, removing the three local re-derivations.
- **Evidence**:

```typescript
// Identical 2-line block in all three files:
type GetMediaFileResponses = ServerInferResponses<typeof contract.mediaFiles.getMediaFile>;
type MediaFileDTO = Extract<GetMediaFileResponses, { status: 200 }>["body"];
```

### Finding 4: `mediaType` parameter typed as `MediaFile["mediaType"]` in one method but `MediaFileDTO["mediaType"]` in another, both from the same enum

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/application/media-files/services/media-files-storage.service.ts:65` and `:262`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `uploadFile` types its `mediaType` parameter as `MediaFile["mediaType"]` (the Drizzle table inferred type, i.e. the `mediaTypeEnum`), which is the correct database-level type. But `uploadChatFile` at line 262 derives `mediaType` as `MediaFileDTO["mediaType"]` from the contract DTO. These two types should be equivalent, but they are arrived at via different paths. If the contract DTO and the DB enum ever diverge (e.g. a new media type added to the DB enum before the contract is regenerated), TypeScript would silently allow a mismatch. Using a single source of truth — the DB enum or a shared contract literal — for the parameter type of `uploadFile` and the local variable in `uploadChatFile` would prevent drift.
- **Suggestion**: Define the allowed media type values once (either from the `mediaTypeEnum` values or a shared contracts literal union) and use that single type consistently across both methods. Since `mediaTypeEnum` is already the DB source of truth, prefer `MediaFile["mediaType"]` everywhere and avoid deriving the type from the DTO shape.
- **Evidence**:

```typescript
// uploadFile — uses DB table type (correct source of truth)
mediaType?: MediaFile["mediaType"];   // line 65

// uploadChatFile — uses DTO contract type (different derivation path)
const mediaType: MediaFileDTO["mediaType"] = isImage ? "image" : isAudio ? "audio" : isVideo ? "video" : "document";
// line 262
```
