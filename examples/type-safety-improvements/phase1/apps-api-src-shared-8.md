# Audit: apps-api-src-shared-8

**Files inspected**: 8
**Findings**: 4

## Summary

The tables in this slice are generally well-typed. `predictions.table.ts` has two untyped jsonb columns that hold important AI output data. `outbox.table.ts` has an untyped `payload` column. The `pending-integrations.table.ts` uses a local `PendingIntegrationStatus` type alias on a `text` column instead of a proper Drizzle enum. The `proactive-notification.table.ts` actions column is typed with a minimal inline type.

## Findings

### Finding 1: `predictedActions` and `contentSignals` in `predictions.table.ts` have no `$type<>` annotations

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/predictions.table.ts:48,38`
- **Category**: missing-strict-typing
- **Impact**: high
- **Description**: Two jsonb columns critical to AI prediction output are untyped:
  - `predictedActions`: described in a comment as "compound action array (1-4 ordered actions)" — should be an array of action types
  - `contentSignals`: described as "detected content signals" — likely a structured detection result

  Both return `unknown` from Drizzle queries. The `predictedAction` (singular) column on line 47 correctly uses `varchar`, but the new `predictedActions` (plural) jsonb column is untyped.

- **Suggestion**: Import or define `PredictedAction[]` (likely correlates with `WorkItemType` union) and a `ContentSignal` type from contracts, then apply `.$type<PredictedAction[]>()` and `.$type<ContentSignal[]>()`.
- **Evidence**:

```typescript
contentSignals: jsonb("content_signals"), // Detected content signals
predictedActions: jsonb("predicted_actions"), // Compound action array (1-4 ordered actions)
```

### Finding 2: `PendingIntegrationStatus` is a local type alias used with `text.$type<>` instead of a Drizzle enum

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/pending-integrations.table.ts:16,32`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: A local `PendingIntegrationStatus = "pending" | "confirmed" | "cancelled" | "expired"` type is defined and applied via `.$type<PendingIntegrationStatus>()` on a `text` column. This is better than bare text, but the type only exists in TypeScript — the database column accepts any string. In `enums.ts`, similar pending statuses use `pgEnum`, which enforces the constraint at the DB level too.
- **Suggestion**: Add a `pendingIntegrationStatusEnum = pgEnum(...)` to `enums.ts` and use it here, following the pattern of `pendingLinkStatusEnum` in the same enums file. This enforces the constraint in PostgreSQL as well.
- **Evidence**:

```typescript
export type PendingIntegrationStatus = "pending" | "confirmed" | "cancelled" | "expired";
// ...
status: text("status").$type<PendingIntegrationStatus>().default("pending").notNull(),
```

### Finding 3: `actions` column in `proactive-notification.table.ts` uses inline anonymous type

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/proactive-notification.table.ts:17`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The `actions` jsonb column is typed with an inline `{ action: string; label: string }[]` type. The `action` field is an untyped string — if this maps to a set of known action identifiers (like the proactive channel or urgency values), it should use a union.
- **Suggestion**: Define a named `ProactiveAction` interface and use a union for the `action` field if the set of values is known. Export the type for use in the application layer.
- **Evidence**:

```typescript
actions: jsonb("actions").$type<{ action: string; label: string }[]>().default([]).notNull(),
```

### Finding 4: `metadata` column in `proactive-notification.table.ts` typed as `Record<string, unknown>`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/proactive-notification.table.ts:26`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The `metadata` jsonb column on proactive notifications is typed as `Record<string, unknown>`, the same weak pattern seen in several other tables. Proactive notifications likely have structured metadata (trigger context, source message ID, etc.).
- **Suggestion**: Define a `ProactiveNotificationMetadata` interface with known optional fields and replace the `Record<string, unknown>` type.
- **Evidence**:

```typescript
metadata: jsonb("metadata").$type<Record<string, unknown>>().default({}).notNull(),
```
