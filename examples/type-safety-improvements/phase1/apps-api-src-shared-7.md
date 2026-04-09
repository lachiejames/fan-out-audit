# Audit: apps-api-src-shared-7

**Files inspected**: 8
**Findings**: 5

## Summary

This slice covers knowledge source infrastructure tables. Multiple jsonb columns lack `$type<>` annotations (`metadataJson`, `warningsJson`, `statsJson`, `sourceConfigJson`, `metricsJson`, etc.), which means these columns return `unknown` from Drizzle queries. Several of these columns have clearly documented shapes in comments or in spec files, making them prime candidates for typing. The `lemon-squeezy-events.table.ts` payload is correctly typed as `Record<string, unknown>` given its open webhook nature.

## Findings

### Finding 1: Multiple untyped jsonb columns in `knowledge-source-revisions.table.ts`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/knowledge-source-revisions.table.ts`
- **Category**: missing-strict-typing
- **Impact**: high
- **Description**: The revisions table has at least 8 jsonb columns with no `$type<>` annotation: `archiveSummaryJson` (line 44), `captureMetadataJson` (line 49), `checkpointJson` (line 51), `estimateJson` (line 57), `importSummaryJson` (line 65), `metricsJson` (line 76), `progressJson` (line 82), `warningsJson` (line 111). All return `unknown` from Drizzle queries. These columns drive SSE progress events and are frequently read in application logic.
- **Suggestion**: Define typed interfaces for each JSON shape (e.g., `RevisionMetrics`, `RevisionProgress`, `RevisionEstimate`, `RevisionCheckpoint`) and apply them via `.$type<>()`. The spec document referenced in the comment (`docs/specs/bulletproof-sync-implementation.md`) likely documents these shapes.
- **Evidence**:

```typescript
archiveSummaryJson: jsonb("archive_summary_json"),
captureMetadataJson: jsonb("capture_metadata_json"),
checkpointJson: jsonb("checkpoint_json"),
estimateJson: jsonb("estimate_json"),
importSummaryJson: jsonb("import_summary_json"),
metricsJson: jsonb("metrics_json"),
progressJson: jsonb("progress_json"),
warningsJson: jsonb("warnings_json"),
```

### Finding 2: `metadataJson` and `warningsJson` in `knowledge-source-items.table.ts` have no `$type<>` annotations

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/knowledge-source-items.table.ts:49,69`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: Both `metadataJson` and `warningsJson` are untyped jsonb columns. `metadataJson` likely contains item-specific metadata (source timestamps, title overrides), while `warningsJson` holds parse warnings. Both are read in the quick-import and archive pipelines.
- **Suggestion**: Define `KnowledgeSourceItemMetadata` and `KnowledgeSourceItemWarning[]` types and apply `.$type<>()`. The warning shape is particularly valuable to type since warnings are surfaced in the UI.
- **Evidence**:

```typescript
metadataJson: jsonb("metadata_json"),
warningsJson: jsonb("warnings_json"),
```

### Finding 3: `statsJson`, `sourceConfigJson`, `warningsJson` in `knowledge-sources.table.ts` have no `$type<>` annotations

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/knowledge-sources.table.ts:41,43,51`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: Three untyped jsonb columns: `statsJson`, `sourceConfigJson`, and `warningsJson`. The `sourceConfigJson` comment says it holds "re-capture binding (e.g., Drive folder ID)" — a very specific shape. `statsJson` presumably tracks item counts and tokens. All return `unknown` from Drizzle.
- **Suggestion**: Define `KnowledgeSourceStats`, `DriveSourceConfig` (or broader `ConnectedSnapshotConfig`), and `KnowledgeSourceWarning[]` types. `sourceConfigJson` is referenced in the import job schema as `sourceConfigOverride: z.record(z.string(), z.unknown())` — aligning these two would be a meaningful improvement.
- **Evidence**:

```typescript
sourceConfigJson: jsonb("source_config_json"),
statsJson: jsonb("stats_json"),
warningsJson: jsonb("warnings_json"),
```

### Finding 4: `estimateJson` and `selectedScopeJson` in `knowledge-source-upload-sessions.table.ts` have no `$type<>` annotations

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/knowledge-source-upload-sessions.table.ts:20,30`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `estimateJson` and `selectedScopeJson` are both untyped jsonb columns. These are populated during the upload preflight flow and are read back to display import estimates and scope to the user. The upload session is a high-traffic read path.
- **Suggestion**: Define `UploadEstimate` and `SelectedScope` interfaces (likely already exist in the application layer given they are shown in the UI) and apply `.$type<>()`.
- **Evidence**:

```typescript
estimateJson: jsonb("estimate_json"),
selectedScopeJson: jsonb("selected_scope_json"),
```

### Finding 5: `metadata` in `knowledge-items.table.ts` typed only as `Record<string, unknown>`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/knowledge-items.table.ts:45`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The `metadata` column on knowledge items is typed as `Record<string, unknown>`. Knowledge items are generated by AI extraction and likely have a consistent metadata shape (source reference, confidence reason, extraction timestamp).
- **Suggestion**: Define a `KnowledgeItemMetadata` interface and apply `.$type<KnowledgeItemMetadata>()`. At minimum, document the expected keys in a JSDoc comment.
- **Evidence**:

```typescript
metadata: jsonb("metadata").$type<Record<string, unknown>>().default(sql`'{}'`).notNull(),
```
