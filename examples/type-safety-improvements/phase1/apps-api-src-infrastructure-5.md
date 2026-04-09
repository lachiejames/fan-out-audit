# Audit: apps-api-src-infrastructure-5

**Files inspected**: 8
**Findings**: 5

## Summary

This batch covers document-intelligence types, the Voyage embedding adapter, embedding service, batch splitter, domain events processor, outbox processor, HTTP exception filter, and async context. The files are largely well-typed. Key findings: (1) the Voyage adapter uses inline structural casts on SDK responses to extract `usage.totalTokens`; (2) a local multimodal content array uses a hand-written inline structural type instead of the SDK-exported type; (3) the domain events processor uses `instance as unknown` to perform a duck-type check; and (4) the outbox processor uses a layered cast chain to extract `error.code`.

## Findings

### Finding 1: Inline structural casts for Voyage SDK response usage in three methods

- **File**: `apps/api/src/infrastructure/embedding/adapters/voyage-embedding.adapter.ts:110,207,325`
- **Category**: type-cast
- **Impact**: medium
- **Description**: In `embed`, `contextualizedEmbed`, and `multimodalEmbed`, the `getTokenUsage` callback casts the result to `{ usage?: { totalTokens?: number } }` to extract token usage. The Voyage AI SDK (`voyageai`) exports typed response interfaces that already include `usage.totalTokens`. Using the SDK's types would eliminate the cast and surface breaking changes at compile time.
- **Suggestion**: Import the relevant response types from `voyageai` (e.g. `EmbeddingsResponse`, `ContextualizedEmbeddingsResponse`, `MultimodalEmbeddingsResponse`) and use them instead of the inline structural casts.
- **Evidence**:

```typescript
// voyage-embedding.adapter.ts:110
getTokenUsage: (result) => {
  const total = (result as { usage?: { totalTokens?: number } })?.usage?.totalTokens ?? 0;
  return total > 0 ? { inputTokens: total, outputTokens: 0, totalTokens: total } : null;
},
```

### Finding 2: Hand-written inline content type in `multimodalEmbed` instead of SDK type

- **File**: `apps/api/src/infrastructure/embedding/adapters/voyage-embedding.adapter.ts:278`
- **Category**: sdk-type-duplication
- **Impact**: medium
- **Description**: The `content` array in `multimodalEmbed` is typed with a hand-written inline interface: `{ type: string; text?: string; imageBase64?: string; imageUrl?: string }[]`. The Voyage AI SDK likely exports a `ContentItem` or similar type for multimodal inputs. The comment notes that the SDK uses Fern serialization, but even if the SDK type differs slightly, a local named alias is clearer than an anonymous inline.
- **Suggestion**: Check the `voyageai` SDK for an exported content item type and use it. If none exists, extract a named `VoyageContentItem` type to the top of the file.
- **Evidence**:

```typescript
const content: {
  type: string;
  text?: string;
  imageBase64?: string;
  imageUrl?: string;
}[] = [];
```

### Finding 3: `instance as unknown` then duck-type cast chain in `DomainEventsProcessor`

- **File**: `apps/api/src/infrastructure/events/domain-events.processor.ts:62,66`
- **Category**: type-cast
- **Impact**: low
- **Description**: `wrapper.instance` is cast to `unknown` then to `{ handle?: unknown }` in a two-step chain to check for the `handle` method. This is the standard NestJS DiscoveryService pattern (instances are typed as `object | null` at best), but the intermediate `unknown` cast is unnecessary. The direct cast `wrapper.instance as { handle?: unknown }` would be cleaner.
- **Suggestion**: Consolidate to a single narrowing check: `const inst = wrapper.instance; if (inst && typeof (inst as { handle?: unknown }).handle === "function")`.
- **Evidence**:

```typescript
const instance = wrapper.instance as unknown;
if (
  instance !== null &&
  typeof instance === "object" &&
  "handle" in instance &&
  typeof (instance as { handle?: unknown }).handle === "function"
) {
```

### Finding 4: Multi-step `error.code` extraction cast chain in `OutboxProcessorService`

- **File**: `apps/api/src/infrastructure/events/outbox-processor.service.ts:170`
- **Category**: type-cast
- **Impact**: low
- **Description**: The outer catch block uses a complex inline expression to extract a schema error code: `error !== null && typeof error === "object" && "code" in error && typeof (error as { code?: unknown }).code === "string" ? (error as { code: string }).code : null`. This multi-step, multi-cast chain is hard to read. A type guard or utility function would be clearer.
- **Suggestion**: Extract `function getErrorCode(err: unknown): string | null` as a local or shared utility that performs this narrowing and returns the code.
- **Evidence**:

```typescript
const schemaErrorCode =
  error !== null &&
  typeof error === "object" &&
  "code" in error &&
  typeof (error as { code?: unknown }).code === "string"
    ? (error as { code: string }).code
    : null;
```

### Finding 5: `payload as Record<string, unknown>` cast when enqueuing outbox events

- **File**: `apps/api/src/infrastructure/events/outbox-processor.service.ts:104`
- **Category**: type-cast
- **Impact**: low
- **Description**: `row.payload as Record<string, unknown>` is cast when constructing the BullMQ job data. The `outboxTable.payload` column is likely typed as `unknown` or `JsonValue` by Drizzle, requiring this cast. However, `DomainEventJob.payload` should be `Record<string, unknown>`, so the cast is functionally correct but silently bypasses any structural mismatch between the stored JSON and the expected job shape.
- **Suggestion**: Type `outboxTable.payload` as `Record<string, unknown>` in the Drizzle schema (if it is always a JSON object) to eliminate the cast at the usage site.
- **Evidence**:

```typescript
{ eventType: row.eventType, outboxId: row.id, payload: row.payload as Record<string, unknown> },
```
