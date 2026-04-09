# Type-Safety Audit: apps-api-src-integrations-17

**Batch 78** — `apps/api/src/integrations/platforms/google-drive/` (search, sync, watch, write services, file-content-extractor utils), `google-gmail/errors/`

**Files inspected**: 6
**Findings**: 6

## Summary

`google-drive-sync.service.ts` has three recurring patterns: `filter(Boolean) as GoogleDriveSyncContext["files"]`, `getEmbeddingTexts(unknown[])` base-class cast, and `fetchThread` returning `Promise<unknown>`. `google-drive-file-content-extractor.ts` has a `resolvedMimeType as (typeof DRIVE_DIRECT_DOWNLOAD_MIME_TYPES)[number]` narrowing cast that is acceptable but could be verified. `google-gmail/errors` is clean. Watch and write services are clean.

## Findings

### Finding 1: filter(Boolean) as GoogleDriveSyncContext["files"] in drive-sync

- **File**: `src/integrations/platforms/google-drive/services/google-drive-sync.service.ts:363`
- **Category**: type-cast
- **Impact**: low
- **Description**: `selection.selected.map(item => fileCache.get(item.id)).filter(Boolean) as GoogleDriveSyncContext["files"]` — `Map.get()` returns `T | undefined`; `filter(Boolean)` narrows away `undefined` but TypeScript needs the cast.
- **Suggestion**: Use `.filter((f): f is GoogleDriveSyncContext["files"][number] => f != null)`.
- **Evidence**:

```typescript
ctx.files = selection.selected.map((item) => fileCache.get(item.id)).filter(Boolean) as GoogleDriveSyncContext["files"];
```

---

### Finding 2: getEmbeddingTexts(parsedItems: unknown[]) in drive-sync

- **File**: `src/integrations/platforms/google-drive/services/google-drive-sync.service.ts:643`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `protected override getEmbeddingTexts(parsedItems: unknown[]): string[]` with `parsedItems as ParsedGoogleDriveFile[]` — same `BaseSyncService` base-class pattern (fourth occurrence across platforms).
- **Suggestion**: Make `BaseSyncService` generic on parsed item type to eliminate this pattern everywhere.
- **Evidence**:

```typescript
protected override getEmbeddingTexts(parsedItems: unknown[]): string[] {
  const files = parsedItems as ParsedGoogleDriveFile[];
  ...
}
```

---

### Finding 3: fetchThread returns Promise<unknown> in drive-sync

- **File**: `src/integrations/platforms/google-drive/services/google-drive-sync.service.ts:668`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `async fetchThread(...): Promise<unknown>` — too-wide return type loses the document shape. The function constructs a well-defined object (id, name, mimeType, etc.) but callers receive `unknown`.
- **Suggestion**: Define a `GoogleDriveThreadResult` interface and use `Promise<GoogleDriveThreadResult | null>`.
- **Evidence**:

```typescript
async fetchThread({ userId, integrationId, threadId }: {
  userId: string;
  integrationId: string;
  threadId: string;
}): Promise<unknown> {
```

---

### Finding 4: resolvedMimeType as (typeof DRIVE_DIRECT_DOWNLOAD_MIME_TYPES)[number] cast

- **File**: `src/integrations/platforms/google-drive/utils/google-drive-file-content-extractor.ts:126`
- **Category**: type-cast
- **Impact**: low
- **Description**: After checking `resolvedMimeType in DIRECT_DOWNLOAD_EXTRACTORS`, the code casts to the tuple element type. The preceding `if` block verified the MIME type is in the object, making this cast structurally sound.
- **Suggestion**: The cast is acceptable here; alternatively use `Object.keys(DIRECT_DOWNLOAD_EXTRACTORS).includes(resolvedMimeType)` which TypeScript can use for better narrowing in some cases.
- **Evidence**:

```typescript
const directDownloadMimeType = resolvedMimeType as (typeof DRIVE_DIRECT_DOWNLOAD_MIME_TYPES)[number];
```

---

### Finding 5: google-drive-watch.service.ts and google-drive-write.service.ts

- **File**: `src/integrations/platforms/google-drive/services/google-drive-watch.service.ts`, `google-drive-write.service.ts`
- **Category**: N/A
- **Impact**: none
- **Description**: Both files are clean — well-typed services using proper `Result<T, E>` returns, typed error factories, and no unsafe casts.
- **Suggestion**: No changes needed.

---

### Finding 6: google-gmail errors file

- **File**: `src/integrations/platforms/google-gmail/errors/google-gmail.errors.ts`
- **Category**: N/A
- **Impact**: none
- **Description**: Clean discriminated union error factory. `GoogleGmailPeopleErrors` follows the same factory pattern. No issues.
- **Suggestion**: No changes needed.
