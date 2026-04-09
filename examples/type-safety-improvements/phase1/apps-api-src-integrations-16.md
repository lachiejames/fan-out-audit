# Type-Safety Audit: apps-api-src-integrations-16

**Batch 77** — `apps/api/src/integrations/platforms/google-docs/` (oauth, search, sync, write services), `google-drive/` (errors, plugin, fetch, oauth services)

**Files inspected**: 8
**Findings**: 5

## Summary

`google-docs-sync.service.ts` has the recurring `getEmbeddingTexts(parsedItems: unknown[])` base-class cast. `google-docs-oauth.service.ts` and `google-drive-oauth.service.ts` both use `extends Record<string, unknown>` on their platform metadata interface. `fetchThread` in `google-docs-sync.service.ts` returns `Promise<unknown>` instead of a typed result. `google-drive.plugin.ts` has bracket-access on preview metadata. All other files are clean.

## Findings

### Finding 1: GoogleDocsPlatformMetadata extends Record<string, unknown>

- **File**: `src/integrations/platforms/google-docs/services/google-docs-oauth.service.ts:26`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `interface GoogleDocsPlatformMetadata extends Record<string, unknown>` adds an index signature that undermines the typed fields `platform`, `email`, `userIdentity`. Same structural issue as `AsanaPlatformMetadata`.
- **Suggestion**: Remove the `Record<string, unknown>` base; cast to `Json` only at the database insertion boundary.
- **Evidence**:

```typescript
interface GoogleDocsPlatformMetadata extends Record<string, unknown> {
  [key: string]: unknown;
  platform: "google-docs";
  email: string;
  userIdentity: { email: string };
}
```

---

### Finding 2: getEmbeddingTexts(parsedItems: unknown[]) in google-docs-sync

- **File**: `src/integrations/platforms/google-docs/services/google-docs-sync.service.ts:553`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `protected override getEmbeddingTexts(parsedItems: unknown[]): string[]` with `parsedItems as ParsedGoogleDoc[]` — same recurring `BaseSyncService` base-class pattern as Asana and GitHub.
- **Suggestion**: Make `BaseSyncService` generic on the parsed item type (same root fix).
- **Evidence**:

```typescript
protected override getEmbeddingTexts(parsedItems: unknown[]): string[] {
  return (parsedItems as ParsedGoogleDoc[]).map((doc) => doc.content);
}
```

---

### Finding 3: fetchThread returns Promise<unknown> in google-docs-sync

- **File**: `src/integrations/platforms/google-docs/services/google-docs-sync.service.ts:591`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `async fetchThread(...): Promise<Result<unknown, IntegrationError>>` — the `unknown` result type loses the structure of the fetched document (id, name, content, etc.), forcing callers to cast.
- **Suggestion**: Define a `GoogleDocsThreadResult` interface and use it as the `Ok` type: `Promise<Result<GoogleDocsThreadResult, IntegrationError>>`.
- **Evidence**:

```typescript
async fetchThread(...): Promise<Result<unknown, IntegrationError>> {
```

---

### Finding 4: GoogleDrivePlatformMetadata extends Record<string, unknown>

- **File**: `src/integrations/platforms/google-drive/services/google-drive-oauth.service.ts:26`
- **Category**: record-weakening
- **Impact**: low
- **Description**: Same issue as Finding 1 — `extends Record<string, unknown>` with redundant `[key: string]: unknown` index signature on a typed metadata interface.
- **Suggestion**: Remove the index signature; cast to `Json` at the DB write boundary only.
- **Evidence**:

```typescript
interface GoogleDrivePlatformMetadata extends Record<string, unknown> {
  [key: string]: unknown;
  platform: "google-drive";
  email: string;
  userIdentity: { email: string };
}
```

---

### Finding 5: Bracket-access on action preview in google-drive.plugin.ts

- **File**: `src/integrations/platforms/google-drive/google-drive.plugin.ts:114`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `preview["recipientName"]`, `preview["recipientContext"]`, `preview["threadContext"]` — bracket access on `preview` typed as `Record<string, string | null | undefined>`. The `typeof` checks (`typeof preview[...] === "string"`) add safety but a typed `UploadFilePreview` interface would be better.
- **Suggestion**: Define a `DriveUploadPreviewFields` interface with typed fields; avoid relying on string-keyed access.
- **Evidence**:

```typescript
const fileName = typeof preview["recipientName"] === "string" ? preview["recipientName"] : "Untitled";
const mimeType = typeof preview["recipientContext"] === "string" ? preview["recipientContext"] : "text/plain";
```
