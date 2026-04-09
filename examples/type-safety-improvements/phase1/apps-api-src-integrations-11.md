# Type-Safety Audit: apps-api-src-integrations-11

**Batch 72** — `apps/api/src/integrations/platforms/asana/` (fetch, oauth, search, sync-fetch, sync, write services, oauth utils, sync utils)

**Files inspected**: 8
**Findings**: 8

## Summary

Recurring `unknown[]` base-class parameter pattern in `asana-sync.service.ts` forces casts to `ResolvedAsanaTask[]`. Multiple casts in asana-fetch and asana-search arise from the `asRecord()` utility returning `Record<string, unknown>`. The `buildTaskMetadata` helper returns `Record<string, unknown>` where a typed interface would be safer. OAuth service has unavoidable `platformMetadata as Partial<AsanaPlatformMetadata>` casts.

## Findings

### Finding 1: err(apiErr as AsanaError) — unnecessary casts in asana-fetch

- **File**: `src/integrations/platforms/asana/services/asana-fetch.service.ts:83`
- **Category**: type-cast
- **Impact**: low
- **Description**: `return err(apiErr as AsanaError)` casts the error result because the client returns a wider error type. If the client `request<T>()` was typed to return `AsanaError` directly this cast would be unnecessary.
- **Suggestion**: Narrow the client's error return type to `AsanaError` so callers don't need to cast.
- **Evidence**:

```typescript
return err(apiErr as AsanaError);
return err(storiesResult.error as AsanaError);
```

---

### Finding 2: platformMetadata as Partial<AsanaPlatformMetadata> in asana-oauth

- **File**: `src/integrations/platforms/asana/services/asana-oauth.service.ts:320`
- **Category**: type-cast
- **Impact**: low
- **Description**: `integration.platformMetadata as Partial<AsanaPlatformMetadata>` is required because `platformMetadata` is typed `unknown` in the database schema. The cast is structurally correct but bypasses validation.
- **Suggestion**: Parse with a Zod schema derived from `AsanaPlatformMetadata` to validate before accessing fields.
- **Evidence**:

```typescript
const metadata = integration.platformMetadata as Partial<AsanaPlatformMetadata>;
```

---

### Finding 3: Multiple as AsanaWorkspace[] casts in asana-search

- **File**: `src/integrations/platforms/asana/services/asana-search.service.ts:228`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `asRecord()` returns `Record<string, unknown>`, forcing every field access to be cast: `as AsanaWorkspace[] | undefined`, `as AsanaProject[]`, etc. This is a structural issue with `asRecord()`.
- **Suggestion**: Replace `asRecord()` with a Zod-validated response type so field access is type-safe without individual casts.
- **Evidence**:

```typescript
const workspaces = responseRecord["workspaces"] as AsanaWorkspace[] | undefined;
const projects = responseRecord["projects"] as AsanaProject[];
```

---

### Finding 4: filter(Boolean) as AsanaTask[] in asana-sync-fetch

- **File**: `src/integrations/platforms/asana/services/asana-sync-fetch.service.ts:449`
- **Category**: type-cast
- **Impact**: low
- **Description**: `.filter(Boolean) as AsanaTask[]` — the `filter(Boolean)` pattern requires a cast because TypeScript can't narrow the element type without a type predicate.
- **Suggestion**: Replace with `.filter((x): x is AsanaTask => x != null)` for type-safe filtering.
- **Evidence**:

```typescript
const tasks = rawTasks.filter(Boolean) as AsanaTask[];
```

---

### Finding 5: getEmbeddingTexts(parsedItems: unknown[]) in asana-sync

- **File**: `src/integrations/platforms/asana/services/asana-sync.service.ts:475`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `protected override getEmbeddingTexts(parsedItems: unknown[]): string[]` — the `BaseSyncService` base class signature forces `unknown[]`, requiring an immediate cast to `ResolvedAsanaTask[]` inside. This is a recurring pattern across all sync services.
- **Suggestion**: Make `BaseSyncService` generic on the parsed item type so overrides receive `T[]` directly. This would eliminate the cast in every sync service implementation.
- **Evidence**:

```typescript
protected override getEmbeddingTexts(parsedItems: unknown[]): string[] {
  return (parsedItems as ResolvedAsanaTask[]).map((task) => buildEmbeddingText(task));
}
```

---

### Finding 6: parsedItems as ResolvedAsanaTask[] in syncAttachments hook

- **File**: `src/integrations/platforms/asana/services/asana-sync.service.ts:584`
- **Category**: type-cast
- **Impact**: low
- **Description**: Same `parsedItems as ResolvedAsanaTask[]` cast appears in the `onContentInserted` hook for the same reason as Finding 5.
- **Suggestion**: Addressed by the same `BaseSyncService` generic refactor in Finding 5.
- **Evidence**:

```typescript
const tasks = parsedItems as ResolvedAsanaTask[];
```

---

### Finding 7: buildTaskMetadata returns Record<string, unknown>

- **File**: `src/integrations/platforms/asana/utils/asana-sync.utils.ts:86`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: `buildTaskMetadata` returns `Record<string, unknown>` even though the returned object has a well-known shape. Callers lose type safety when accessing fields from the metadata.
- **Suggestion**: Define an `AsanaTaskMetadata` interface matching the return shape and update the function signature.
- **Evidence**:

```typescript
function buildTaskMetadata(task: AsanaTask): Record<string, unknown> {
  return {
    assigneeEmail: task.assignee?.email ?? null,
    completedAt: task.completed_at ?? null,
    dueOn: task.due_on ?? null,
    projectIds: task.projects?.map((p) => p.gid) ?? [],
    ...
  };
}
```

---

### Finding 8: asana-write.service.ts

- **File**: `src/integrations/platforms/asana/services/asana-write.service.ts`
- **Category**: N/A
- **Impact**: none
- **Description**: Clean service — well-typed with proper `Result<T, E>` returns and no unsafe casts.
- **Suggestion**: No changes needed.
