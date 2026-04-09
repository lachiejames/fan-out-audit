# Audit: apps-api-src-domain-5

**Files inspected**: 8
**Findings**: 4

## Summary

The service ports in this batch are well-structured. Key findings: `subscriptionStatus` is duplicated as a literal union across multiple port interfaces; `BackgroundJobsPort.enqueueWebhookJob` accepts `data: unknown`; `IIndexingBudgetPort.getMonthlyUsageByPlatform` returns `Record<string, number>` where the key is a platform ID; and `IAIWorkItemGenerationPort` duplicates the `subscriptionStatus` union from `IAIModelPort`.

## Findings

### Finding 1: `subscriptionStatus` literal union duplicated across multiple ports

- **File**: `apps/api/src/domain/ports/services/ai-model.port.ts:32`, `apps/api/src/domain/ports/services/ai-work-item-generation.port.ts:43`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: The literal union `"trialing" | "active" | "past_due" | "cancelled" | "expired" | "paused"` is defined inline in both `CreateClaudeModelParams.subscriptionStatus`, `CreateClaudeModelWithToolsParams.subscriptionStatus`, and `WorkItemGenerationConfig.subscriptionStatus`. If the set of subscription statuses changes (e.g., a new `"frozen"` status is added), all three must be updated independently. The contracts package likely exports `SubscriptionStatus` which should be used instead.
- **Suggestion**: Import `SubscriptionStatus` from `@slopweaver/contracts` (already imported in `billing.port.ts` as `SubscriptionStatus`) and replace all three inline literals.
- **Evidence**:

```typescript
// ai-model.port.ts:32
subscriptionStatus?: "trialing" | "active" | "past_due" | "cancelled" | "expired" | "paused";

// ai-work-item-generation.port.ts:43
subscriptionStatus?: "trialing" | "active" | "past_due" | "cancelled" | "expired" | "paused";
```

### Finding 2: `BackgroundJobsPort.enqueueWebhookJob` accepts `data: unknown`

- **File**: `apps/api/src/domain/ports/services/background-jobs.port.ts:101`
- **Category**: any-usage
- **Impact**: medium
- **Description**: `enqueueWebhookJob(params: { name: string; data: unknown; jobId?: string })` uses `data: unknown` because webhook payloads are platform-specific (Slack, Gmail, Linear). However, this means callers can enqueue any value including primitives, `null`, or non-serializable objects. At minimum `data: Record<string, unknown>` would ensure the payload is a JSON-serializable object, which is required by BullMQ.
- **Suggestion**: Change `data: unknown` to `data: Record<string, unknown>` to enforce that webhook payloads are JSON objects, as BullMQ requires.
- **Evidence**:

```typescript
abstract enqueueWebhookJob(params: { name: string; data: unknown; jobId?: string }): Promise<void>;
```

### Finding 3: `IIndexingBudgetPort.getMonthlyUsageByPlatform` returns `Record<string, number>` with undocumented key semantics

- **File**: `apps/api/src/domain/ports/services/indexing-budget.port.ts:45`
- **Category**: record-weakening
- **Impact**: low
- **Description**: The method returns `Record<string, number>` where the keys are platform IDs. Using `string` as the key type means callers cannot tell at compile time whether they should look up by `"google-gmail"` or `"gmail"` or some internal identifier. If `IntegrationPlatform` or `PlatformId` from `@slopweaver/contracts` is the correct key type, it should be used.
- **Suggestion**: Return `Partial<Record<IntegrationPlatform, number>>` or `Record<string, number>` with a JSDoc `@returns {Record<platformId, messageCount>}` comment clarifying the key format.
- **Evidence**:

```typescript
abstract getMonthlyUsageByPlatform(params: {
  userId: string;
}): Promise<Result<Record<string, number>, IndexingBudgetPortError>>;
```

### Finding 4: `IAIModelPort.logger` typed as `unknown` in `CreateClaudeModelParams`

- **File**: `apps/api/src/domain/ports/services/ai-model.port.ts:25-26`
- **Category**: any-usage
- **Impact**: low
- **Description**: `logger?: unknown` in `CreateClaudeModelParams` is intentionally opaque (the comment says "Infrastructure adapters may cast to the expected logger type"). While the architectural reasoning is valid, the `unknown` type means any value (number, string, etc.) is accepted without a type error, which could mask bugs where the wrong value is passed.
- **Suggestion**: Use `logger?: ILoggerPort` by importing the domain logger port (which is already in the domain layer). This adds type safety without violating layer boundaries.
- **Evidence**:

```typescript
/**
 * Logger is intentionally opaque to avoid importing NestJS types into domain.
 * Infrastructure adapters may cast to the expected logger type.
 */
logger?: unknown;
```
