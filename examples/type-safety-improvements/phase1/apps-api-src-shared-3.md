# Audit: apps-api-src-shared-3

**Files inspected**: 8
**Findings**: 4

## Summary

Queue constant files for slice 3 follow the same clean Zod pattern as slice 2, but several issues appear: multiple platform-specific cursor state type aliases that are all identical to the base `Record<string, unknown>`, an untyped `jsonb` column (`payload`) in the knowledge-source-import-events table, and a `Record<number, number>` key type in `behavioral-fingerprint.table.ts` that is nominally safe but semantically undocumented.

## Findings

### Finding 1: `CursorState` and all platform-specific cursor aliases are all `Record<string, unknown>`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/sync-checkpoints.table.ts:18-34`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: `CursorState = Record<string, unknown>` is defined as the base type, and then 15 platform-specific aliases (`GmailCursorState`, `SlackCursorState`, etc.) all equal `CursorState` exactly. These type aliases provide zero type safety — they are all `Record<string, unknown>` and are structurally identical. A developer using `GmailCursorState` gets no compile-time guarantee about shape.
- **Suggestion**: Define concrete types for each platform's cursor (e.g., `GmailCursorState = { historyId: string; nextPageToken?: string }`) or remove the aliases and use `CursorState` everywhere with a JSDoc note that platform-specific code should validate with Zod at runtime. The aliases as they stand give a false sense of type safety.
- **Evidence**:

```typescript
export type CursorState = Record<string, unknown>;
export type GmailCursorState = CursorState;
export type SlackCursorState = CursorState;
// ... (15 identical aliases)
```

### Finding 2: `payload` column in `knowledge-source-import-events.table.ts` has no `$type<>` annotation

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/knowledge-source-import-events.table.ts:24`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: The `payload` jsonb column is declared without a `$type<>` annotation, meaning it returns `unknown` from Drizzle queries. Unlike similar columns in other tables (e.g., `outbox.table.ts` payload), no explicit type is given. The table represents SSE progress events which have a known shape.
- **Suggestion**: Add `.$type<KsImportProgressEvent>()` (or whatever the event payload union is). If the type lives in a contracts package, import and apply it. At minimum, add `.$type<Record<string, unknown>>()` with a comment noting the specific expected shapes.
- **Evidence**:

```typescript
payload: jsonb("payload").notNull(),
```

### Finding 3: `outbox.table.ts` — `payload` column has no `$type<>` annotation

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/outbox.table.ts:36`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: The `payload` jsonb column on the outbox table is declared without a `$type<>` annotation, returning `unknown` from Drizzle. This is the transactional outbox table used for all domain events — a particularly high-value target for typing since processors fan out to many event types.
- **Suggestion**: Define a `DomainEventPayload` union type (or import it if it already exists as part of the domain events infrastructure) and annotate `payload: jsonb("payload").$type<DomainEventPayload>().notNull()`.
- **Evidence**:

```typescript
payload: jsonb("payload").notNull(),
```

### Finding 4: `PushNotificationResult.urgency` typed as `string` instead of a union

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/constants/queues/push-notifications.constants.ts:24-29`
- **Category**: record-weakening
- **Impact**: low
- **Description**: The `PushNotificationResult` interface has both `channel: string` and `urgency: string`. These correspond to enum values (`pushChannelEnum`, `pushUrgencyTierEnum`) defined in `enums.ts`. Using bare `string` misses the opportunity to catch typos or drift.
- **Suggestion**: Import `PushChannel` and `PushUrgencyTier` from enums and use them: `channel: PushChannel; urgency: PushUrgencyTier`.
- **Evidence**:

```typescript
export interface PushNotificationResult {
  channel: string;
  sent: boolean;
  skipped?: boolean;
  urgency: string;
}
```
