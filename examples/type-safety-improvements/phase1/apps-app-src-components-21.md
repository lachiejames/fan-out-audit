# Audit: apps-app-src-components-21

**Files inspected**:

- `apps/app/src/components/review-signals/review-signals-strip.tsx`
- `apps/app/src/components/settings/settings-profile.tsx`
- `apps/app/src/components/settings/data-security-card.tsx`
- `apps/app/src/components/settings/delete-account-modal.tsx`
- `apps/app/src/components/settings/knowledge-sources/knowledge-source-card.tsx`
- `apps/app/src/components/settings/knowledge-sources/knowledge-source-detail-dialog.tsx`
- `apps/app/src/components/settings/knowledge-sources/knowledge-source-imported-knowledge.tsx`
- `apps/app/src/components/settings/knowledge-sources/knowledge-source-provenance.tsx`

**Findings**: 4

---

## Summary

Four findings. `knowledge-source-card.tsx` and `knowledge-source-detail-dialog.tsx` both define `PROVIDER_LABELS: Record<string, string>` from the same `KNOWLEDGE_SOURCE_PROVIDER_META` source — a clear duplicate derivation. `knowledge-source-detail-dialog.tsx` also uses `Record<string, React.ReactNode>` for `ITEM_TYPE_ICONS` where a narrower key union is available. `knowledge-source-imported-knowledge.tsx` uses `Record<string, string>` for `CATEGORY_COLORS` and `STATUS_BADGES` maps where `ImportedKnowledgeCategory` / `ImportedKnowledgeStatus` types from contracts would be correct keys. `knowledge-source-provenance.tsx` casts `sourceConfig` and `captureMetadata` to inline structural types (`as { folderName?: string; ... }`) instead of contract types.

---

## Findings

### Finding 1: Duplicate `PROVIDER_LABELS` derivation in two knowledge-source files

- **File**: `apps/app/src/components/settings/knowledge-sources/knowledge-source-card.tsx` (line 29) and `apps/app/src/components/settings/knowledge-sources/knowledge-source-detail-dialog.tsx` (line 47)
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: Both files independently compute:
  ```typescript
  const PROVIDER_LABELS: Record<string, string> = Object.fromEntries(
    Object.entries(KNOWLEDGE_SOURCE_PROVIDER_META).map(([k, v]) => [k, v.label]),
  );
  ```
  This is identical derivation logic duplicated in two files. If `KNOWLEDGE_SOURCE_PROVIDER_META` changes (new provider, label rename), both constants must be updated.
- **Suggestion**: Extract this derivation to a single shared constant — either in a shared utility file (e.g., `apps/app/src/shared/utils/knowledge-source-labels.ts`) or as a named export from the module that imports `KNOWLEDGE_SOURCE_PROVIDER_META`. Both files then import the shared constant.

---

### Finding 2: `ITEM_TYPE_ICONS: Record<string, React.ReactNode>` should use a narrower key type

- **File**: `apps/app/src/components/settings/knowledge-sources/knowledge-source-detail-dialog.tsx`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `ITEM_TYPE_ICONS` at line 51 uses `Record<string, React.ReactNode>`. The keys (`"conversation"`, `"document_section"`, `"memory_blob"`, `"project_document"`) are `KnowledgeItemType` values from `@slopweaver/contracts`. Using `Partial<Record<KnowledgeItemType, React.ReactNode>>` would make the map exhaustive and flag stale or misspelled keys.
- **Suggestion**: Import `KnowledgeItemType` (or equivalent) from `@slopweaver/contracts` and use `Partial<Record<KnowledgeItemType, React.ReactNode>>`.
- **Evidence**:
  ```typescript
  // line 51
  const ITEM_TYPE_ICONS: Record<string, React.ReactNode> = {
    conversation: <MessageSquare className="h-4 w-4" />,
    document_section: <FileText className="h-4 w-4" />,
    ...
  };
  ```

---

### Finding 3: `CATEGORY_COLORS` and `STATUS_BADGES` use `Record<string, ...>` instead of contract key types

- **File**: `apps/app/src/components/settings/knowledge-sources/knowledge-source-imported-knowledge.tsx`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: `CATEGORY_COLORS` (line 7) and `STATUS_BADGES` (line 16) are both `Record<string, ...>`. The keys correspond to `ImportedKnowledgeItemResponse` category and status fields from `@slopweaver/contracts`. Using contract-derived types for the keys would make these maps exhaustive.
- **Suggestion**:
  - For `CATEGORY_COLORS`: use `Partial<Record<ImportedKnowledgeCategory, string>>` where `ImportedKnowledgeCategory` is the union of knowledge item categories from contracts.
  - For `STATUS_BADGES`: use `Partial<Record<ImportedKnowledgeStatus, { className: string; label: string }>>` where `ImportedKnowledgeStatus` is the union of item review statuses from contracts.
- **Evidence**:
  ```typescript
  // line 7
  const CATEGORY_COLORS: Record<string, string> = {
    context: "...",
    fact: "...",
    ...
  };
  // line 16
  const STATUS_BADGES: Record<string, { className: string; label: string }> = {
    active: { ... },
    archived: { ... },
    pending_review: { ... },
  };
  ```

---

### Finding 4: Inline structural casts for `sourceConfig` and `captureMetadata` in `knowledge-source-provenance.tsx`

- **File**: `apps/app/src/components/settings/knowledge-sources/knowledge-source-provenance.tsx`
- **Category**: type-cast
- **Impact**: medium
- **Description**: At lines 21–22, `source.sourceConfig` and `activeRevision.captureMetadata` are cast to inline structural types:
  ```typescript
  const config = source.sourceConfig as { folderName?: string; folderId?: string; includeSubfolders?: boolean };
  const metadata = activeRevision?.captureMetadata as { fileCount?: number; capturedAt?: string } | null;
  ```
  And again at line 46 for Notion metadata. If `KnowledgeSourceResponse.sourceConfig` is typed as `unknown` or `Record<string, unknown>` in contracts, these casts are the minimal fix. However, if contracts define typed `sourceConfig` shapes per provider, the casts are bypassing that.
- **Suggestion**: Check whether `@slopweaver/contracts` exports typed `sourceConfig` shapes for `"drive_folder"` and Notion export providers. If they exist, use discriminated union narrowing instead of inline casts. If the contracts genuinely use `unknown` here, document the cast with a comment referencing the contract type.
- **Evidence**:
  ```typescript
  // line 21
  const config = source.sourceConfig as { folderName?: string; folderId?: string; includeSubfolders?: boolean };
  // line 46
  const metadata = activeRevision.captureMetadata as {
    pageCount?: number;
    databaseCount?: number;
    capturedAt?: string;
  };
  ```
