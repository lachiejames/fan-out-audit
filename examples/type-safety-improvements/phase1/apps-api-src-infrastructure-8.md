# Audit: apps-api-src-infrastructure-8

**Files inspected**: 8
**Findings**: 4

## Summary

This batch covers the remaining queue service files (pattern-extraction, prediction-generation, push-notifications, queue-module factory, queue-service factory, scheduled-jobs, webhooks, work-item-generation). Most files are thin factory wrappers with no findings. The `queue-module.factory.ts` has the most substantive issues.

## Findings

### Finding 1: `any[]` in `createQueueModule` `queueService` parameter type

- **File**: `apps/api/src/infrastructure/queues/queue-module.factory.ts:34`
- **Category**: any-usage
- **Impact**: medium
- **Description**: The `queueService` parameter accepts `Type<QueueServiceInstance<TJob>> | (new (...args: any[]) => { queue: { close(): Promise<void> } })`. The second branch of the union uses `any[]` constructor args. Inside `GeneratedQueueModule`, the service is injected via `@Inject(queueService)` which is typed `QueueServiceInstance<TJob>`. The `any[]` in the union allows passing a class that does not actually implement `QueueServiceInstance<TJob>`, which would be caught at runtime rather than compile time.
- **Suggestion**: Remove the second union branch and require `Type<QueueServiceInstance<TJob>>`. If the factory needs to support classes that only have `queue.close()` (without the full `QueueServiceInstance` shape), create a separate interface and use it consistently.
- **Evidence**:

```typescript
queueService: Type<QueueServiceInstance<TJob>> | (new (...args: any[]) => { queue: { close(): Promise<void> } });
```

### Finding 2: `exports: any[]` in `createQueueModule`

- **File**: `apps/api/src/infrastructure/queues/queue-module.factory.ts:43`
- **Category**: any-usage
- **Impact**: low
- **Description**: `const exports: any[]` is used to build the NestJS module exports array. NestJS module metadata is typed as `Array<Type | DynamicModule | ...>`, so `any[]` could be replaced with the appropriate NestJS union type.
- **Suggestion**: Use the NestJS `ExportedProvider` / `ModuleExportOptions` types (or simply `(Type | DynamicModule)[]`) from `@nestjs/common` to avoid the `any[]`.
- **Evidence**:

```typescript
const exports: any[] = exportBullModule ? [BullModule, queueService] : [queueService];
```

### Finding 3: `WebhooksQueueService` missing job type parameter (cross-reference)

- **File**: `apps/api/src/infrastructure/queues/webhooks-queue.service.ts:3`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: Already noted in Batch 57 Finding 2, but this file is in the present batch scope too. `createQueueService({ queueName: WEBHOOKS_QUEUE })` defaults to `T = unknown`. The `Queue<unknown>.add()` accepts any payload.
- **Suggestion**: Pass the webhook job union type as generic parameter.
- **Evidence**:

```typescript
export const WebhooksQueueService = createQueueService({ queueName: WEBHOOKS_QUEUE });
```

### Finding 4: Scheduled-jobs queue uses typed generic but `ScheduledJobData` may not cover all job payloads

- **File**: `apps/api/src/infrastructure/queues/scheduled-jobs-queue.service.ts:3`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `createQueueService<ScheduledJobData>({ queueName: SCHEDULED_JOBS_QUEUE })` passes a typed generic, which is correct. However, if `ScheduledJobData` is a wide type (e.g. `{ scheduledAt: string }` without discriminating between different scheduled job types), callers cannot distinguish between job kinds at compile time. This is not directly visible from this file, but worth flagging for review of the constants file.
- **Suggestion**: Verify that `ScheduledJobData` in `scheduled-jobs.constants` is a discriminated union or narrow enough to be useful. If all scheduled jobs share the same payload, the current approach is fine.
- **Evidence**:

```typescript
export const ScheduledJobsQueueService = createQueueService<ScheduledJobData>({
  queueName: SCHEDULED_JOBS_QUEUE,
});
```
