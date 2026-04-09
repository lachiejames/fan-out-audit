# Audit: apps-api-src-shared-2

**Files inspected**: 8
**Findings**: 3

## Summary

The queue constants files are consistently well-typed using Zod schemas with `z.infer<>` for job payloads. Two issues stand out: a `Record<string, unknown>` on the domain-events payload schema that is intentionally open but could carry a comment, a `Record<string, unknown>` optional field in the knowledge-source-import job schema, and a `HIGH_FREQUENCY_JOB_TYPES` typed as `ReadonlySet<string>` in the scheduled-jobs constants when a stricter union would be possible.

## Findings

### Finding 1: `domainEventJobSchema` payload typed as `z.record(z.string(), z.unknown())`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/constants/queues/domain-events.constants.ts:8`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: The `payload` field on `domainEventJobSchema` uses `z.record(z.string(), z.unknown())`, which means downstream processors receive `Record<string, unknown>` for all event payloads. Each event type has a specific shape (e.g., `UserRegisteredEvent`), but that shape is not enforced at the queue boundary. This makes processors do their own runtime parsing with no schema-level help.
- **Suggestion**: Consider defining a discriminated union schema per domain event type, or at minimum use `z.unknown()` (a single value) so the intent — "parse this yourself" — is explicit. The current `record` typing implies a flat object when events are often nested.
- **Evidence**:

```typescript
payload: z.record(z.string(), z.unknown()),
```

### Finding 2: `sourceConfigOverride` typed as `z.record(z.string(), z.unknown())`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/constants/queues/knowledge-source-import.constants.ts:11`
- **Category**: record-weakening
- **Impact**: low
- **Description**: The `sourceConfigOverride` field is typed as `Record<string, unknown>` via `z.record(z.string(), z.unknown())`. The comment says it is used for Drive re-capture config, which presumably has a known shape (`{ folderId: string }` or similar). Callers pass untyped blobs.
- **Suggestion**: Import and use the actual `sourceConfigJson` shape from the knowledge sources schema, or define a named type for Drive re-capture config and use it here. At minimum add a JSDoc comment naming the expected shape.
- **Evidence**:

```typescript
sourceConfigOverride: z.record(z.string(), z.unknown()).optional(),
```

### Finding 3: `HIGH_FREQUENCY_JOB_TYPES` typed as `ReadonlySet<string>` instead of `ReadonlySet<ScheduledJobType>`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/constants/queues/scheduled-jobs.constants.ts:73`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `HIGH_FREQUENCY_JOB_TYPES` is a `ReadonlySet<string>` containing two values from `SCHEDULED_JOB_TYPES`. Callers do `has()` checks against `job.name` (a string), which is correct at runtime, but the set itself has no awareness of the union. Using `ReadonlySet<(typeof SCHEDULED_JOB_TYPES)[keyof typeof SCHEDULED_JOB_TYPES]>` would make membership checks type-aware.
- **Suggestion**: Define a `ScheduledJobType = (typeof SCHEDULED_JOB_TYPES)[keyof typeof SCHEDULED_JOB_TYPES]` alias (or import the one from contracts if it exists) and type the set as `ReadonlySet<ScheduledJobType>`.
- **Evidence**:

```typescript
export const HIGH_FREQUENCY_JOB_TYPES: ReadonlySet<string> = new Set([
  SCHEDULED_JOB_TYPES.OUTBOX_PROCESS,
  SCHEDULED_JOB_TYPES.INTEGRATION_ACTION_OUTBOX_PROCESS,
]);
```
