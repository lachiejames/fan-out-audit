# Audit: apps-api-src-application-19

**Files inspected**: 8
**Findings**: 6

## Summary

These files implement the knowledge-source import pipeline: two parsers (Notion ZIP export, website HTML snapshot) and six services covering archive parsing, progress pub/sub, run lifecycle, source lifecycle, preflight estimation, and quick-import orchestration. The most significant type-safety gaps are unsafe `as` casts on untyped JSONB columns, an unsafe `parsed.payload` passthrough to a typed handler without validation, a `Record<string, KnowledgeSourceProvider>` that should be a const-keyed mapped type, a `phase: string | null` that could be a union literal, and a repeated `as KnowledgeSourceProvider` cast that is avoidable.

## Findings

### Finding 1: `getCheckpoint` casts raw JSONB to `Record<string, unknown>` without narrowing

- **File**: `apps/api/src/application/knowledge-sources/services/knowledge-source-import-run.service.ts:102`
- **Category**: type-cast
- **Impact**: medium
- **Description**: The `checkpointJson` column is `jsonb` in the Drizzle schema, so Drizzle infers it as `unknown`. The return value is force-cast with `as Record<string, unknown>` rather than being narrowed or typed at the schema level. Any consumer calling `getCheckpoint` then receives `Record<string, unknown> | null` and must re-cast every field it reads, spreading the unsafety downstream.
- **Suggestion**: Type the column in the Drizzle table definition via `jsonb("checkpoint_json").$type<Record<string, unknown>>()` so the ORM returns the correct type without any cast. Alternatively, add a narrowing guard: `if (typeof row.checkpointJson !== 'object' || row.checkpointJson === null) return null;` before returning. Either approach eliminates the cast.
- **Evidence**:

```typescript
// knowledge-source-import-run.service.ts:94-103
async getCheckpoint({ revisionId }: { revisionId: string }): Promise<Record<string, unknown> | null> {
  const rows = await this.database.db
    .select({ checkpointJson: knowledgeSourceRevisionsTable.checkpointJson })
    ...
  const row = rows[0];
  return (row?.checkpointJson as Record<string, unknown>) ?? null;  // unsafe cast
}
```

---

### Finding 2: `metadataJson` cast to `Record<string, unknown>` repeated in two places

- **File**: `apps/api/src/application/knowledge-sources/services/knowledge-source-quick-import.service.ts:130` and `:289`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `metadataJson` is also an untyped `jsonb` column (Drizzle infers `unknown`). The same `as Record<string, unknown>` cast appears twice in `knowledge-source-quick-import.service.ts` when mapping DB items to rankable/extraction-input shapes, and again in `knowledge-source-response.mappers.ts:129` (a third occurrence in a nearby file). This is a recurring unsafe cast that could be fixed once at the schema layer.
- **Suggestion**: Add `.$type<Record<string, unknown>>()` to the `metadataJson` column definition in `apps/api/src/shared/db/tables/knowledge-source-items.table.ts`. This removes all three casts simultaneously and ensures every caller gets the correct type.
- **Evidence**:

```typescript
// knowledge-source-quick-import.service.ts:128-131
metadata: (item.metadataJson as Record<string, unknown>) ?? null,

// knowledge-source-quick-import.service.ts:287-290
metadata: (entry.dbItem.metadataJson as Record<string, unknown>) ?? null,
```

---

### Finding 3: Redis pub/sub message passthrough delivers `unknown` to a typed handler without validation

- **File**: `apps/api/src/application/knowledge-sources/services/knowledge-source-import-progress.service.ts:143-150`
- **Category**: missing-strict-typing
- **Impact**: high
- **Description**: The `messageHandler` function parses raw Redis JSON and calls `handler(parsed.payload)` directly. `JSON.parse` returns `any`, so `parsed.payload` is `any` — it is silently assignable to the typed `ImportProgressPayload` parameter of the handler callback. If the Redis message is malformed or a different shape, the handler receives bad data with no runtime or compile-time error.
- **Suggestion**: Validate the parsed payload before passing it to the handler. Either use a Zod schema (`ImportProgressPayloadSchema.safeParse(parsed.payload)`) or add a type guard `isImportProgressPayload(parsed.payload)`. The `ImportProgressPayload` interface is already defined in the same file — converting it to a Zod schema would provide both validation and the TypeScript type for free.
- **Evidence**:

```typescript
// knowledge-source-import-progress.service.ts:143-150
const messageHandler = (_ch: string, message: string): void => {
  try {
    const parsed = JSON.parse(message); // parsed is `any`
    handler(parsed.payload); // silently typed as ImportProgressPayload — no validation
  } catch {
    // Ignore parse errors
  }
};
```

---

### Finding 4: `getEventHistory` return type uses `payload: unknown` instead of `ImportProgressPayload`

- **File**: `apps/api/src/application/knowledge-sources/services/knowledge-source-import-progress.service.ts:188`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `getEventHistory` returns `Array<{ createdAt: Date; eventType: string; id: string; payload: unknown }>`. The `payload` column is the same JSONB column that stores `ImportProgressPayload`. Returning `unknown` forces every caller to cast or re-validate data they can reasonably trust. The `eventType` field is also typed as `string` here, even though `ImportProgressPayload.eventType` is a known union `"progress" | "completed" | "failed" | "cancelled" | "paused"`.
- **Suggestion**: Type the payload column with `.$type<ImportProgressPayload>()` in the `knowledge-source-import-events.table.ts` schema, then update the return type to reflect `payload: ImportProgressPayload`. Also tighten `eventType` to the same union used in `ImportProgressPayload`.
- **Evidence**:

```typescript
// knowledge-source-import-progress.service.ts:188
}): Promise<Array<{ createdAt: Date; eventType: string; id: string; payload: unknown }>> {

// knowledge-source-import-events.table.ts:24 (Drizzle schema)
payload: jsonb("payload").notNull(),  // no .$type<>() annotation
```

---

### Finding 5: `MIME_TO_PROVIDER` uses `Record<string, KnowledgeSourceProvider>` — keys are open-ended

- **File**: `apps/api/src/application/knowledge-sources/services/knowledge-source-preflight.service.ts:19`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `MIME_TO_PROVIDER` is a const object with exactly five known MIME-type string keys. Typing it as `Record<string, KnowledgeSourceProvider>` means the lookup at line 56 (`MIME_TO_PROVIDER[mimeType]`) returns `KnowledgeSourceProvider` rather than `KnowledgeSourceProvider | undefined`, hiding the potential miss. The code then uses `?? null` to handle the miss, but TypeScript doesn't know the lookup can return `undefined` — meaning the `??` is a code smell that the type is hiding.
- **Suggestion**: Use `satisfies` with a stricter type or declare the object as `const` without an explicit type annotation and let inference produce the narrow record type. A `Partial<Record<string, KnowledgeSourceProvider>>` would at least make the undefined-on-miss explicit. Even better: derive keys from a literal union of the supported MIME types.
- **Evidence**:

```typescript
// knowledge-source-preflight.service.ts:18-26
/** Map MIME types to provider guesses (without buffer inspection). */
const MIME_TO_PROVIDER: Record<string, KnowledgeSourceProvider> = {
  "application/json": "json",
  "application/pdf": "pdf",
  "application/vnd.openxmlformats-officedocument.wordprocessingml.document": "docx",
  "text/csv": "csv",
  "text/markdown": "md",
  "text/plain": "txt",
};
// ...
return MIME_TO_PROVIDER[mimeType] ?? null; // TS thinks this can never be null
```

---

### Finding 6: `providerHint as KnowledgeSourceProvider` cast avoidable with a type guard or validated input

- **File**: `apps/api/src/application/knowledge-sources/services/knowledge-source-preflight.service.ts:45`
- **Category**: type-cast
- **Impact**: low
- **Description**: `guessProvider` accepts `providerHint?: string` and immediately casts it to `KnowledgeSourceProvider` without validation. If the caller supplies an invalid string (e.g., a typo from a client request), the cast silently produces an invalid `KnowledgeSourceProvider` value that propagates into `buildPreflightEstimate`.
- **Suggestion**: Change the parameter type of `providerHint` to `KnowledgeSourceProvider | undefined` (since callers should already be validating at the controller layer) — or add a runtime check using `PLATFORM_IDS` / a Zod enum to validate before casting. At minimum, the `runPreflight` public method should accept `providerHint?: KnowledgeSourceProvider` instead of `providerHint?: string` to push the type safety to the boundary.
- **Evidence**:

```typescript
// knowledge-source-preflight.service.ts:42-45
  providerHint?: string;
}): KnowledgeSourceProvider | null {
  if (providerHint) return providerHint as KnowledgeSourceProvider;  // unvalidated cast
```
