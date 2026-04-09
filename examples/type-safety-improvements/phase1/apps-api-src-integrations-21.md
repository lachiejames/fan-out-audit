# Type Safety Audit — apps/api/src/integrations (Batch 21)

**Files inspected**: 8  
**Findings**: 7

## Summary

The Jira service files contain several recurring type-safety issues. The most structurally significant is the generic REST client (`jira-client.provider.ts`) which must cast `response.json() as T` — this is inherent to the untyped fetch API but creates a hole across every endpoint. The OAuth service casts `platformMetadata as JiraPlatformMetadata | null` in two places without runtime validation. The sync service uses `parsedItems as ResolvedIssue[]` in two `BaseSyncService` overrides, and a `.filter(Boolean) as JiraApiIssue[]` pattern instead of a proper type guard. The ADF parser performs repeated inline casts from `unknown` that could be replaced with a typed recursive traversal.

## Findings

---

### Finding 1

**File**: `apps/api/src/integrations/platforms/jira/providers/jira-client.provider.ts:246`  
**Category**: Type cast (`as T`)  
**Impact**: Medium

**Description**: The `get<T>`, `post<T>`, `put<T>`, and `delete<T>` methods of `JiraApiClient` all cast `(await response.json()) as T`. Because `fetch` returns `any`, every caller gets back whatever type they ask for with no runtime validation. A malformed or unexpected API response will satisfy the cast and produce downstream errors.

**Suggestion**: Document that `T` must be validated by the caller using a Zod schema. Alternatively, provide typed wrappers (e.g., `getValidated<T>(path, schema: z.ZodType<T>)`) that parse and validate the response body.

**Evidence**:

```typescript
// jira-client.provider.ts:246
return ok((await response.json()) as T);

// jira-client.provider.ts:285
return ok({} as T); // cast for 204 No Content

// jira-client.provider.ts:288
return ok((await response.json()) as T);
```

---

### Finding 2

**File**: `apps/api/src/integrations/platforms/jira/services/jira-oauth.service.ts:448`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `buildMasterTokenData` casts `integration.platformMetadata` (typed `unknown`) to `JiraPlatformMetadata | null` without runtime validation. If the stored JSONB is stale or missing required fields (`cloudId`, `cloudName`, `cloudUrl`), the cast succeeds silently and subsequent property access returns `undefined`.

**Suggestion**: Parse `platformMetadata` with the `JiraPlatformMetadata` Zod schema (if one exists) or a type guard before use.

**Evidence**:

```typescript
// jira-oauth.service.ts:448
const metadata = integration.platformMetadata as JiraPlatformMetadata | null;
// jira-oauth.service.ts:473 (same pattern in buildMasterUserInfo)
const metadata = integration.platformMetadata as JiraPlatformMetadata | null;
```

---

### Finding 3

**File**: `apps/api/src/integrations/platforms/jira/services/jira-sync.service.ts:396`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `ctx.resumeState` is assigned via `nextResumeState as JiraCursorState`. The `nextResumeState` object is built locally but has type `Record<string, unknown>`, so the cast is effectively unchecked.

**Suggestion**: Replace the cast with a `satisfies JiraCursorState` assertion during object construction so TypeScript validates the shape at the assignment site.

**Evidence**:

```typescript
// jira-sync.service.ts:396
ctx.resumeState = nextResumeState as JiraCursorState;
```

---

### Finding 4

**File**: `apps/api/src/integrations/platforms/jira/services/jira-sync.service.ts:405`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `.filter(Boolean) as JiraApiIssue[]` is used to narrow the result of `issueCache.get(item.id)` (which returns `JiraApiIssue | undefined`). While the cast achieves the correct type, a proper type guard (`.filter((x): x is JiraApiIssue => x !== undefined)`) makes the intent explicit and is safer if the map value type changes.

**Suggestion**: Use a typed filter predicate: `.filter((x): x is JiraApiIssue => x !== undefined)`.

**Evidence**:

```typescript
// jira-sync.service.ts:405
ctx.allIssues = selection.selected.map((item) => issueCache.get(item.id)).filter(Boolean) as JiraApiIssue[];
```

---

### Finding 5

**File**: `apps/api/src/integrations/platforms/jira/services/jira-sync.service.ts:624`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `getEmbeddingTexts` and `onContentInserted` both receive `parsedItems: unknown[]` (the `BaseSyncService` abstract method signature) and immediately cast to `ResolvedIssue[]`. There is no validation that items are actually `ResolvedIssue` instances.

**Suggestion**: Add a type guard `isResolvedIssue(item: unknown): item is ResolvedIssue` and filter/assert before processing, or widen the `BaseSyncService` generic to thread the concrete item type through.

**Evidence**:

```typescript
// jira-sync.service.ts:624-625
protected override getEmbeddingTexts(parsedItems: unknown[]): string[] {
  return (parsedItems as ResolvedIssue[]).map((r) => r.content);
}

// jira-sync.service.ts:655
const resolvedIssues = parsedItems as ResolvedIssue[];
```

---

### Finding 6

**File**: `apps/api/src/integrations/platforms/jira/utils/jira-adf-parser.utils.ts:33`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `parseAdfToText` casts `adf` to structured inline types three times rather than defining a shared `AdfNode` type. The repeated casts from `unknown` make the parsing logic harder to follow and the inline shapes could diverge silently.

**Suggestion**: Define an `AdfNode` interface capturing `{ type?: string; text?: string; content?: unknown[]; attrs?: { url?: string } }` and use it consistently. Replace inline casts with a single upfront narrowing using the type guard pattern.

**Evidence**:

```typescript
// jira-adf-parser.utils.ts:33-42
const doc = adf as { type?: string; content?: unknown[] };
// ...
if (Array.isArray((adf as { content?: unknown[] }).content)) {
  return extractTextFromNodes({ nodes: (adf as { content: unknown[] }).content });
}

// jira-adf-parser.utils.ts:62-67
const n = node as {
  type?: string;
  text?: string;
  content?: unknown[];
  attrs?: { url?: string };
};
```

---

### Finding 7

**File**: `apps/api/src/integrations/platforms/jira/utils/jira-custom-fields.utils.ts:70`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: After a `typeof value === "object" && value !== null` check, the code casts to `Record<string, unknown>`. TypeScript already narrows `unknown` to `object` after this check, but `object` doesn't support index signatures. The cast is technically safe here but could be replaced with `value as Record<PropertyKey, unknown>` or by using `Object.prototype.hasOwnProperty`.

**Suggestion**: Use `value as Record<string, unknown>` (current practice is fine) but add a comment explaining why the cast is safe post-narrowing, to prevent future over-eager refactors.

**Evidence**:

```typescript
// jira-custom-fields.utils.ts:70
const obj = value as Record<string, unknown>;
```
