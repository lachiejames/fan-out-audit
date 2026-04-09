# Audit: apps-api-src-shared-13

**Files inspected**: 8
**Findings**: 5

## Summary

This slice contains several tables with JSONB columns that are typed as `Record<string, unknown>` or `Record<string, unknown>[]` when concrete types already exist in the codebase (e.g. `ActionPreview`, `ReviewSignal`, `EditOperation`). Two webhook tables also use plain `text` columns for fields that carry well-known string literal sets, and `webhook-registrations.table.ts` uses `text` for `status` and `registrationType` columns without `$type<>` narrowing.

## Findings

### Finding 1: `aiSources` and `targetIdentifiers` in `work-items.table.ts` typed as `Record<string, unknown>`

- **File**: `apps/api/src/shared/db/tables/work-items.table.ts:37,78`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `aiSources` is typed `Record<string, unknown>[]` and `targetIdentifiers` as `Record<string, unknown>`. Both fields carry structured data: `targetIdentifiers` is documented as a "canonical typed identifiers" discriminated union. The contracts layer already exports `PlatformIdentifiers` — that type (or a union that includes it) should be used instead of the opaque record.
- **Suggestion**: Use `.$type<PlatformIdentifiers>()` (or the appropriate discriminated union from `@slopweaver/contracts`) for `targetIdentifiers`. For `aiSources`, define a concrete `AiSource` interface (e.g. `{ contentId: string; score: number }`) if the shape is stable.
- **Evidence**:
  ```ts
  aiSources: jsonb("ai_sources").$type<Record<string, unknown>[]>(),
  targetIdentifiers: jsonb("target_identifiers").$type<Record<string, unknown>>(),
  ```

### Finding 2: `details` in `work-item-events.table.ts` typed as `Record<string, unknown>`

- **File**: `apps/api/src/shared/db/tables/work-item-events.table.ts:19`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `details` is typed as `Record<string, unknown>`. Because events are typed by `auditEventTypeEnum`, each event type likely has a predictable details shape. A per-event-type discriminated union would make read-sites safer.
- **Suggestion**: Define per-event-type detail interfaces and a discriminated union (e.g. `WorkItemEventDetails`) keyed on `auditEventTypeEnum` values, then apply `.$type<WorkItemEventDetails>()`.
- **Evidence**:
  ```ts
  details: jsonb("details").$type<Record<string, unknown>>(),
  ```

### Finding 3: `externalResult` in `work-item-runs.table.ts` typed as `Record<string, unknown>`

- **File**: `apps/api/src/shared/db/tables/work-item-runs.table.ts:20`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `externalResult` stores the platform API result of a work item execution. Each platform's result shape is likely known (e.g. Gmail send response, Slack post response). A more specific type would improve safety.
- **Suggestion**: Define a `WorkItemExternalResult` type in `shared/types/work-items.types.ts` (even as a partial or discriminated union) and use it in the column.
- **Evidence**:
  ```ts
  externalResult: jsonb("external_result").$type<Record<string, unknown>>().default({}).notNull(),
  ```

### Finding 4: `status` and `registrationType` plain `text` columns in `webhook-registrations.table.ts`

- **File**: `apps/api/src/shared/db/tables/webhook-registrations.table.ts:56-57`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: Both `status` (`'active', 'expired', 'failed'`) and `registrationType` (`'webhook', 'poll'`) are stored as plain `text` without a `$type<>` annotation. The comments document the valid values, but TypeScript sees these as `string`, allowing invalid values to be written without a compile-time error.
- **Suggestion**: Apply `.$type<"active" | "expired" | "failed">()` to `status` and `.$type<"webhook" | "poll">()` to `registrationType`, or define a pgEnum for both.
- **Evidence**:
  ```ts
  registrationType: text("registration_type").notNull().default("webhook"),
  status: text("status").notNull().default("active"),
  ```

### Finding 5: `WorkflowNode.data` typed as `Record<string, unknown>` in `workflows.table.ts`

- **File**: `apps/api/src/shared/db/tables/workflows.table.ts:18`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The `WorkflowNode` interface has `data: Record<string, unknown>`, which mirrors React Flow's own generic `data` property. If the codebase uses React Flow's typed node variants, the data type could be tightened to a union of known node data shapes.
- **Suggestion**: If node types are known (e.g. `"trigger" | "action" | "condition"`), define a discriminated union of data shapes and replace `Record<string, unknown>` with it.
- **Evidence**:
  ```ts
  export interface WorkflowNode {
    data: Record<string, unknown>;
  }
  ```

## No additional findings

`users.table.ts`, `vip-contact.table.ts`, `webhook-events.table.ts`, and `webhook-event-deliveries.table.ts` are clean with no `any`, no unsafe casts, and proper enum column usage.
