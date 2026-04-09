# Audit: apps-api-src-shared-10

**Files inspected**: 8
**Findings**: 4

## Summary

The final slice covers suggestion feedback/presentation/suppression tables, summaries, and sync infrastructure. The `sync-checkpoints.table.ts` `cursorState` column is typed as `CursorState = Record<string, unknown>` (the same alias weakness flagged in slice 3). `sync-runs.table.ts` status column reappears here with a known-values comment but plain text type. Several jsonb columns in `suggestion-feedback` and `suggestion-presentations` are untyped. The `summaries.table.ts` is clean.

## Findings

### Finding 1: `cursorState` in `sync-checkpoints.table.ts` typed as `CursorState = Record<string, unknown>`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/sync-checkpoints.table.ts:51`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: The `cursorState` jsonb column uses `.$type<CursorState>()` where `CursorState = Record<string, unknown>`. The column stores per-platform resumption tokens (Gmail historyId, Slack oldest, Linear cursor). These are platform-specific structures with known shapes but all aliased to the same opaque `Record<string, unknown>`. Reading `checkpoint.cursorState` gives no type information.
- **Suggestion**: This is a cross-cutting concern already reported in slice 3 (Finding 1 there covers the type aliases). To fix here specifically: define a discriminated union `type CheckpointCursorState = GmailCursor | SlackCursor | LinearCursor | ...` with concrete fields, and apply `.$type<CheckpointCursorState>()`. The `platform` column on the same table provides the discriminant.
- **Evidence**:

```typescript
cursorState: jsonb("cursor_state").$type<CursorState>().notNull().default({}),
```

### Finding 2: `metadata` column in `insights.table.ts` typed as `Record<string, unknown>`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/insights.table.ts:21`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: The `metadata` jsonb column on the insights table is typed as `Record<string, unknown>`. Insights are observability events with a specific `eventType` (from `insightEventTypeEnum`). Each event type likely has a structured payload (e.g., `knowledge_extraction_started` includes a `contentId`, `semantic_search_query` includes a query string). The `Record<string, unknown>` type is too broad.
- **Suggestion**: Define a discriminated union `InsightMetadata` keyed by `insightEventTypeEnum` values, similar to how `AuditRagContext` and `AuditToolInvocation` are typed in `llm-calls.table.ts`. Import and apply `.$type<InsightMetadata>()`.
- **Evidence**:

```typescript
metadata: jsonb("metadata").$type<Record<string, unknown>>().default({}).notNull(),
```

### Finding 3: `jobData` in `failed-jobs.table.ts` has no `$type<>` annotation

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/failed-jobs.table.ts:19`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The `jobData` jsonb column stores the full BullMQ job data payload when a job fails permanently. It has no `$type<>` annotation and returns `unknown`. Given that all queue job schemas are defined as Zod schemas in the constants files, typing this as a union of all known job types would enable typed post-mortem analysis.
- **Suggestion**: Define `type AnyJobData = AccountDeletionJob | AttachmentEmbeddingJob | IntegrationSyncJob | ...` (union of all queue job types) and apply `.$type<AnyJobData>()`. Alternatively, use a minimal base type like `{ userId?: string; [key: string]: unknown }` with a JSDoc comment.
- **Evidence**:

```typescript
jobData: jsonb("job_data").notNull(),
```

### Finding 4: `lemon-squeezy-events.table.ts` `payload` typed as `Record<string, unknown>` — acceptable but worth noting

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/lemon-squeezy-events.table.ts:18`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The Lemon Squeezy webhook event payload is typed as `$type<Record<string, unknown>>()`. The Lemon Squeezy SDK (`@lemonsqueezy/lemonsqueezy.js`) exports typed webhook payload interfaces. Using the SDK types here would provide compile-time safety when processing these events in the billing webhook handler.
- **Suggestion**: Import the Lemon Squeezy webhook payload type from the SDK (e.g., `LemonsqueezyWebhookPayload`) and apply `.$type<LemonsqueezyWebhookPayload>()`. If the SDK types are too broad or change frequently, a local interface mirroring the used fields is a reasonable alternative.
- **Evidence**:

```typescript
payload: jsonb("payload").$type<Record<string, unknown>>().notNull(),
```
