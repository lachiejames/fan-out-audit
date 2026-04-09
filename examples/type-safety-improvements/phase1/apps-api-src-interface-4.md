# Type-Safety Audit — Batch 99

**Files audited:**

- `apps/api/src/interface/agents/tools/slack/send-slack-message.review-edit.ts`
- `apps/api/src/interface/agents/tools/workspace/edit-knowledge.tool.ts`
- `apps/api/src/interface/agents/tools/_manifests.ts`
- `apps/api/src/interface/agents/tools/workspace/search-workspace.tool.ts`
- `apps/api/src/interface/agents/tools/workspace/toggle-vip.tool.ts`
- `apps/api/src/interface/agents/tools/workspace/update-todo.tool.ts`
- `apps/api/src/interface/agents/utils/context-formatting.utils.ts`
- `apps/api/src/interface/agents/utils/prompt-generation.utils.ts`

---

## Findings

### 1. edit-knowledge.tool.ts — record-weakening + type-cast

**Lines:** 61–67
**Category:** record-weakening, type-cast
**Severity:** medium
**Description:** `const data: Record<string, unknown> = {}` is built with known fields (`content`, `confidence`, `category`) and then cast `data as Parameters<typeof knowledgeService.updateKnowledge>[0]["data"]` to match the service signature. The intermediate `Record<string, unknown>` is unnecessary — the data object could be typed directly.
**Code:**

```typescript
const data: Record<string, unknown> = {};
if (input.content !== undefined) data["content"] = input.content;
// ...
await knowledgeService.updateKnowledge({
  data: data as Parameters<typeof knowledgeService.updateKnowledge>[0]["data"],
```

**Fix:** Type `data` as `Partial<{ content: string; confidence: number; category: string }>` and build it with proper typed fields. Pass it directly without the cast.

---

### 2. edit-knowledge.tool.ts — missing-strict-typing (result value access)

**Lines:** 93–97
**Category:** missing-strict-typing
**Severity:** low
**Description:** `result.value` from `knowledgeService.updateKnowledge` is accessed with bracket notation `updated["category"]`, `updated["confidence"]`, etc., treating the result as a record. The `updateKnowledge` service return type should be a concrete `Knowledge` entity type.
**Code:**

```typescript
const updated = result.value;
return {
  knowledge: {
    category: updated["category"],
    confidence: updated["confidence"],
```

**Fix:** Ensure `KnowledgeService.updateKnowledge` returns `Result<Knowledge, KnowledgeError>` where `Knowledge` is the typed entity, and access fields directly (`updated.category`, `updated.confidence`).

---

### 3. search-workspace.tool.ts — missing-strict-typing (platforms array)

**Line:** 115
**Category:** missing-strict-typing
**Severity:** low
**Description:** `const integrationPlatforms = platforms?.filter((p) => p !== "knowledge")` — the result is `string[]` but these are integration platform IDs that should be typed as `PlatformId[]` from `@slopweaver/contracts` after filtering.
**Code:**

```typescript
const integrationPlatforms = platforms?.filter((p) => p !== "knowledge");
```

**Fix:** Add a type assertion after filtering: `as PlatformId[]` or use a type guard to validate platform IDs against `PLATFORM_IDS` from `@slopweaver/contracts`.

---

### 4. context-formatting.utils.ts — record-weakening

**Line:** 54
**Category:** record-weakening
**Severity:** low
**Description:** `EnrichedContext.metadata: Record<string, unknown>` — the enriched context metadata is a weakly typed bag. The metadata is used in `buildContextBlock` for display only, but callers that set metadata could benefit from a typed structure.
**Code:**

```typescript
export interface EnrichedContext {
  metadata: Record<string, unknown>;
```

**Fix:** Define a `EnrichedContextMetadata` interface with known fields (`from?: string`, `subject?: string`, `platform?: string`, etc.) and extend with an index signature for additional fields: `{ [key: string]: unknown } & EnrichedContextMetadata`.

---

## No Findings

- `send-slack-message.review-edit.ts` — Same `ToolReviewEditMapper` boundary pattern tracked in batch 97/98; no additional findings here.
- `_manifests.ts` — Clean manifest aggregation file; imports and spreads typed `ToolManifestEntry[]`; no findings.
- `toggle-vip.tool.ts` — Clean tool with typed Zod schema and explicit service interface; no findings.
- `update-todo.tool.ts` — Clean tool using typed `UpdateTodoData` from the `TodosPort`; no findings.
- `prompt-generation.utils.ts` — Pure string utility functions with explicit types; no findings.
