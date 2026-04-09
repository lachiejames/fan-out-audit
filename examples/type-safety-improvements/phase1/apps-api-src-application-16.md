# Audit: apps-api-src-application-16

**Files inspected**: 8
**Findings**: 3

## Summary

These files cover three domain event handlers (OAuth callback, sync content indexed, webhook error) and four feedback-loop services (pattern confidence decay, pattern extraction job orchestration, AI-powered writing pattern extraction, and writing pattern CRUD). The handlers share a recurring pattern of casting `job.payload` (typed as `Record<string, unknown>`) to specific payload types without runtime validation. The extraction service contains an unnecessary type cast when reading `usage` from the Vercel AI SDK's `generateText` result, whose return type already exposes `usage` as a well-typed `LanguageModelUsage`.

## Findings

### Finding 1: `job.payload` cast to specific payload types without narrowing (three handlers)

- **File**: `apps/api/src/application/events/handlers/oauth-callback.handler.ts:43`
- **File**: `apps/api/src/application/events/handlers/sync-content-indexed.handler.ts:51`
- **File**: `apps/api/src/application/events/handlers/webhook-error.handler.ts:41`
- **Category**: type-cast
- **Impact**: high
- **Description**: `DomainEventJob.payload` is typed as `Record<string, unknown>` (from the Zod schema in `domain-events.constants.ts`). All three handlers immediately cast it to a specific contract payload type with `as SomePayload` and then destructure fields directly. If the wrong event type is routed to a handler (or the payload schema drifts), this silently produces `undefined` fields at runtime with no TypeScript protection. The `as` cast suppresses all type errors that would otherwise catch mismatches.
- **Suggestion**: The payload type should be narrowed at the boundary rather than assumed. Two options: (a) Use Zod to parse the payload against the expected schema at handler entry (e.g. `OAuthCallbackCompletedPayloadSchema.parse(job.payload)`) — this gives runtime safety plus correct TS types without the cast; or (b) if parsing overhead is undesirable, at minimum replace `as SomePayload` with a type predicate guard so TypeScript tracks the narrowing explicitly. The contract schemas are already available in `@slopweaver/contracts`.
- **Evidence**:

```typescript
// oauth-callback.handler.ts:43
const payload = job.payload as OAuthCallbackCompletedPayload;

// sync-content-indexed.handler.ts:51
const payload = job.payload as SyncContentIndexedPayload;

// webhook-error.handler.ts:41
const payload = job.payload as WebhookProcessingFailedPayload;
```

---

### Finding 2: Unnecessary `unknown` cast for `generateText` result's `usage` field

- **File**: `apps/api/src/application/feedback-loops/services/writing-pattern-extraction.service.ts:270`
- **Category**: type-cast
- **Impact**: medium
- **Description**: After calling `generateText(...)` from the Vercel AI SDK, the code checks `typeof result === "object" && "usage" in result` and then casts with `(result as { usage?: unknown }).usage` to extract the usage object. However, `generateText` returns `GenerateTextResult<...>` which has a `readonly usage: LanguageModelUsage` property — already fully typed. The `LanguageModelUsage` type includes `inputTokens`, `outputTokens`, `inputTokenDetails`, and `outputTokenDetails`. The defensive unknown-cast and in-check are dead code that obscure the true type and make the usage extraction unnecessarily complex.
- **Suggestion**: Remove the runtime check and cast entirely. Use `result.usage` directly, and either pass it as `LanguageModelUsage` to `extractInputOutputTokensFromUsage`, or update that utility to accept `LanguageModelUsage` instead of `unknown` — which would also improve safety there.
- **Evidence**:

```typescript
// writing-pattern-extraction.service.ts:270-273
const usage =
  typeof result === "object" && result !== null && "usage" in result
    ? (result as { usage?: unknown }).usage
    : undefined;
const { cacheCreationInputTokens, cacheReadInputTokens, inputTokens, outputTokens } = extractInputOutputTokensFromUsage(
  { usage },
);
```

The SDK type at the call site:

```typescript
// ai SDK index.d.ts
readonly usage: LanguageModelUsage;  // always present on GenerateTextResult
```

---

### Finding 3: `record-weakening` — `DomainEventJob.payload` typed as `Record<string, unknown>`

- **File**: `apps/api/src/shared/constants/queues/domain-events.constants.ts:8`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: The `domainEventJobSchema` uses `z.record(z.string(), z.unknown())` for `payload`, which infers as `Record<string, unknown>`. Because all three handlers cast this to specific payload types (see Finding 1), the weak `Record` type is the root cause of the unsafety. The actual valid payloads are a closed set defined in `@slopweaver/contracts` (`OAuthCallbackCompletedPayload`, `SyncContentIndexedPayload`, `WebhookProcessingFailedPayload`, and others). The current schema provides no discrimination between event types.
- **Suggestion**: Define the payload as a discriminated union tied to `eventType`, or at minimum change `payload` to `z.unknown()` and parse it per-handler at the point of use (making the cast visible and explicit). Ideally, create a typed union schema: `z.discriminatedUnion("eventType", [...perEventSchemas])` so TypeScript narrows `payload` automatically once `eventType` is checked. This would eliminate all three `as SomePayload` casts in the handlers.
- **Evidence**:

```typescript
// domain-events.constants.ts:5-11
export const domainEventJobSchema = z.object({
  eventType: z.string().min(1),
  outboxId: z.uuid(),
  payload: z.record(z.string(), z.unknown()), // ← too wide
});

export type DomainEventJob = z.infer<typeof domainEventJobSchema>;
// payload: Record<string, unknown>
```
