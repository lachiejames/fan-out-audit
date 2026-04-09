# Audit: apps-api-src-application-20

**Files inspected**: 8
**Findings**: 5

## Summary

These files implement the knowledge-source domain: response mapping from DB rows to DTOs, revision/archive/quick-import status management, upload session lifecycle, batch planning, archive chunking and selection, and duplicate suppression. The code is generally clean but carries several type-cast patterns that bypass the type system, two instances of inline string-literal unions that duplicate already-exported enums, and one convoluted `updatePayload` type that forces a double cast to paper over an intentional dynamic structure.

---

## Findings

### Finding 1: `updateArchiveStatus` inlines string literals that duplicate exported enums

- **File**: `apps/api/src/application/knowledge-sources/services/knowledge-source-revision-ops.service.ts:190-194`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: The `status` and `phase` parameters are typed with hand-written string-literal unions. These exact union types are already exported from `apps/api/src/shared/db/tables/enums.ts` as `ArchiveStatus` (`"not_started" | "processing" | "completed" | "failed"`) and `ArchivePhase` (`"selecting" | "chunking" | "embedding" | "finalizing"`). The same file already imports `QuickImportStatus` and `QuickImportPhase` from `@slopweaver/contracts` for the parallel `updateQuickImportStatus` method — `ArchiveStatus`/`ArchivePhase` should be treated the same way. If the DB enum ever gains a new variant the hand-written union silently falls out of sync.
- **Suggestion**: Import `ArchiveStatus` and `ArchivePhase` from `@/shared/db/tables/enums` (or re-export them from `@slopweaver/contracts` if contracts already contains them) and replace the inline literals.
- **Evidence**:

```typescript
async updateArchiveStatus({
  revisionId,
  status,
  phase,
  progressPercent,
  importMode,
}: {
  revisionId: string;
  status: "not_started" | "processing" | "completed" | "failed";  // ← duplicates ArchiveStatus enum
  phase?: "selecting" | "chunking" | "embedding" | "finalizing" | null;  // ← duplicates ArchivePhase enum
  progressPercent?: number | null;
  importMode?: "manual" | "searchable_archive";  // ← duplicates KnowledgeSourceImportMode enum
```

---

### Finding 2: `updatePayload` uses a conditional type to produce `Record<string, unknown>` then double-casts to `$inferInsert`

- **File**: `apps/api/src/application/knowledge-sources/services/knowledge-sources.service.ts:369-385`
- **Category**: type-cast
- **Impact**: medium
- **Description**: The update payload is typed via an obscure conditional type expression (`Parameters<typeof …>[0] extends infer _T ? Record<string, unknown> : never`) purely to allow dynamic key spreading, and then cast again to `typeof knowledgeSourceRevisionsTable.$inferInsert` at the call site. Neither cast is validated; a typo in a field name (e.g. `cancelledAt` vs the actual column name) would compile silently. The real type is available directly from the table's inferred insert type.
- **Suggestion**: Replace the conditional-type annotation and the final cast with a direct `Partial<typeof knowledgeSourceRevisionsTable.$inferInsert>` variable and build it with explicit spreads. This eliminates both casts and gives exact column-level type checking.
- **Evidence**:

```typescript
const updatePayload: Parameters<typeof this.database.db.update>[0] extends infer _T ? Record<string, unknown> : never =
  {
    status,
    ...(phase !== undefined && { phase }),
    // …
  };

const [updated] = await this.database.db
  .update(knowledgeSourceRevisionsTable)
  .set(updatePayload as typeof knowledgeSourceRevisionsTable.$inferInsert); // ← cast hides the real type
```

---

### Finding 3: JSONB→DTO casts bypass type narrowing in `knowledge-source-response.mappers.ts`

- **File**: `apps/api/src/application/knowledge-sources/services/knowledge-source-response.mappers.ts:67-74`, `:49`, `:83`, `:129`
- **Category**: type-cast
- **Impact**: medium
- **Description**: Multiple JSONB columns are cast directly to contract types with `as SomeType | null` without any runtime narrowing. Drizzle types JSONB columns as `unknown`, so these casts silently succeed even if the stored JSON is corrupt or from an older schema version. The same pattern repeats for `metricsJson`, `warningsJson`, `importSummaryJson`, `archiveSummaryJson`, `estimateJson`, `sourceConfigJson`, `captureMetadataJson`, and `metadataJson`. If the DB contains malformed data the contract type guarantee is false at runtime.
- **Suggestion**: Either add a Zod parse at the mapper boundary (using the contract schemas which presumably have Zod schemas already) or, at minimum, replace bare `as T` with an assertion function (`assertIs<T>`) so failures are caught in tests rather than silently propagating to clients.
- **Evidence**:

```typescript
const metricsRaw = revision.metricsJson as KnowledgeSourceMetrics | null;
const warningsRaw = (revision.warningsJson ?? []) as KnowledgeSourceWarning[];
const importSummaryRaw = revision.importSummaryJson as KnowledgeSourceImportSummary | null;
const archiveSummaryRaw = revision.archiveSummaryJson as ArchiveSummary | null;
const estimateRaw = revision.estimateJson as PreflightEstimate | null;

// …and separately:
sourceConfig: (source.sourceConfigJson as Record<string, unknown>) ?? null,
captureMetadata: (revision.captureMetadataJson as Record<string, unknown>) ?? null,
metadata: (item.metadataJson as Record<string, unknown>) ?? null,
```

---

### Finding 4: `uploadMethod` cast in `toResponse` due to untyped `text()` column

- **File**: `apps/api/src/application/knowledge-sources/services/knowledge-source-upload-session.service.ts:332`
- **Category**: type-cast
- **Impact**: low
- **Description**: `session.uploadMethod` is stored as a plain `text()` Drizzle column and cast to `"standard" | "resumable"` at mapping time. If someone writes an unexpected value to the DB (or a future enum variant) the cast succeeds silently. The parallel `selectedScope` field has the same cast pattern on line 330.
- **Suggestion**: Convert `uploadMethod` to a pgEnum in the table schema (an `uploadMethodEnum` with `["standard", "resumable"]`) so Drizzle infers the narrow type automatically, eliminating the cast. `selectedScope` is intentionally open JSON so `UploadSessionResponse["selectedScope"]` is acceptable there.
- **Evidence**:

```typescript
uploadMethod: session.uploadMethod as "standard" | "resumable",
// column definition in the table:
uploadMethod: text("upload_method").default("standard").notNull(),
```

---

### Finding 5: `ArchiveSelectionResult.disposition` is a wide string union that partially overlaps with the DB's `archiveDisposition` column type

- **File**: `apps/api/src/application/knowledge-sources/utils/knowledge-source-archive-selection.utils.ts:41-43`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: `ArchiveSelectionResult` defines `disposition` as a local string-literal union `"selected" | "skipped_short" | "skipped_duplicate" | "skipped_boilerplate"`. This is likely the same set of values written to `knowledgeSourceItemsTable.archiveDisposition`. If the two sets diverge (e.g. a new skipped variant is added to the DB enum but not here, or vice-versa) the compiler will not catch it. Checking the enum in `enums.ts` would confirm whether a shared type already exists.
- **Suggestion**: Confirm whether an `archiveDispositionEnum` exists in `apps/api/src/shared/db/tables/enums.ts`. If so, derive `ArchiveSelectionResult["disposition"]` from `(typeof archiveDispositionEnum.enumValues)[number]` rather than re-declaring the union. If not, create the enum so both the DB column and the util share one source of truth.
- **Evidence**:

```typescript
/** Result of archive selection for a single item. */
export type ArchiveSelectionResult = {
  disposition: "selected" | "skipped_short" | "skipped_duplicate" | "skipped_boilerplate";
  item: ArchiveSelectableItem;
};
```
