# Type-Safety Audit: apps-api-src-integrations-5

**Batch 66** — `apps/api/src/integrations/core/services/` (base-sync-lifecycle, base-sync, context-indexing, files-api, indexing-budget, live-search, meeting-sync, oauth-base.callback)

## Summary

8 files reviewed. Two high-priority findings: a type cast mutation of a context field in `base-sync.service.ts`, and a DB string cast to a narrower union in `files-api.service.ts`. The `oauth-base.callback.ts` `unknown` token flow is intentionally generic. The `live-search.service.ts` reduce initializer cast is minor.

---

## Findings

### 1

- **File**: `apps/api/src/integrations/core/services/base-sync-lifecycle.utils.ts`
- **Line**: 176
- **Category**: Type casts
- **Impact**: Low
- **Description**: `phase as Parameters<typeof buildProgressEvent>[1]` — string cast to extract a phase type. The phase parameter is already a `string` but needs narrowing to the exact union accepted by `buildProgressEvent`.
- **Suggestion**: Annotate the `phase` parameter with the exact union type `Parameters<typeof buildProgressEvent>[1]` at the function signature level to eliminate the cast.
- **Evidence**: `buildProgressEvent(ctx, phase as Parameters<typeof buildProgressEvent>[1], metrics);`

---

### 2

- **File**: `apps/api/src/integrations/core/services/base-sync.service.ts`
- **Line**: 343
- **Category**: Type casts
- **Impact**: High
- **Description**: `(ctx as { onProgress?: SyncProgressCallback }).onProgress = trackedOnProgress` — casts the context to inject a field that TypeScript doesn't know exists. This bypasses type safety to mutate a read-only-ish context shape.
- **Suggestion**: Add `onProgress?: SyncProgressCallback` to the `BaseSyncContext` interface so the assignment is type-safe without casting.
- **Evidence**: `(ctx as { onProgress?: SyncProgressCallback }).onProgress = trackedOnProgress;`

---

### 3

- **File**: `apps/api/src/integrations/core/services/base-sync.service.ts`
- **Lines**: 246, 360
- **Category**: Type casts
- **Impact**: Low
- **Description**: Repeated `phase as Parameters<typeof buildProgressEvent>[1]` cast — same pattern as finding 1 but appears in two places in `base-sync.service.ts`.
- **Suggestion**: Same fix as finding 1: annotate phase parameters properly in `BaseSyncService` methods.
- **Evidence**: Pattern repeated at two call sites.

---

### 4

- **File**: `apps/api/src/integrations/core/services/files-api.service.ts`
- **Line**: 215
- **Category**: Type casts
- **Impact**: Medium
- **Description**: `file.purpose as FilePurpose` — the `purpose` column is stored as a `text` in DB (via Drizzle). Casting to the narrower `FilePurpose` union without validation means stale or unexpected DB values become silently wrong at runtime.
- **Suggestion**: Add a Zod parse or an `isFilePurpose` type guard at the DB boundary to validate before casting.
- **Evidence**: `const purpose: FilePurpose = file.purpose as FilePurpose;`

---

### 5

- **File**: `apps/api/src/integrations/core/services/live-search.service.ts`
- **Line**: ~120
- **Category**: Type casts
- **Impact**: Low
- **Description**: `groupByPlatform` uses `{} as Record<string, Integration[]>` cast in the `reduce` initializer. TypeScript cannot infer the accumulator type here but the cast could be replaced with an explicit type annotation on the `reduce` call.
- **Suggestion**: Use `reduce<Record<string, Integration[]>>((acc, integration) => { ... }, {})` to avoid the cast.
- **Evidence**: `const grouped = integrations.reduce((acc, integration) => { ... }, {} as Record<string, Integration[]>);`

---

### 6

- **File**: `apps/api/src/integrations/core/services/meeting-sync.service.ts`
- **Lines**: 164–168
- **Category**: Unsafe `unknown`
- **Impact**: Low
- **Description**: `(providerError as { message: unknown }).message` — necessary because `TranscriptionProvider<TError = unknown>` defaults `TError` to `unknown` (see Batch 64, finding 2). The root cause is the loose generic default.
- **Suggestion**: Fix `TranscriptionProvider` to default `TError` to `Error` (Batch 64, finding 2), which would eliminate this cast.
- **Evidence**: `const msg = (providerError as { message: unknown }).message;`

---
