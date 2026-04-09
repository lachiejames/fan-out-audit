# Type Safety Audit — apps/api/src/integrations (Batch 28)

**Files inspected**: 8  
**Findings**: 5

## Summary

Batch 28 covers the Microsoft Teams plugin/actions/search/sync and the Monday.com errors/plugin/client-provider. The Teams plugin and sync service are generally clean. The Monday client provider's `query<T>` method casts `response.json() as MondayGraphQLResponse<T>` — inherent to the GraphQL client pattern but unchecked. The Monday plugin and sync service use `platformMetadata as { accountId?: string } | null` without validation. Monday's `fetchThread` returns `Promise<unknown>` like the OneDrive equivalent.

## Findings

---

### Finding 1

**File**: `apps/api/src/integrations/platforms/microsoft-teams/microsoft-teams.plugin.ts:190` (approximate)  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `parseCursorState({ value: params.resumeState }) as MicrosoftTeamsCursorState` — the same unvalidated cursor cast found in all plugin files. This is the sixth instance of this pattern.

**Suggestion**: A shared `parseCursorStateAs<T>(value: unknown, schema: z.ZodType<T>): T` utility would eliminate all six instances.

**Evidence**:

```typescript
// microsoft-teams.plugin.ts (syncMessages)
syncParams.resumeState = parseCursorState({ value: params.resumeState }) as MicrosoftTeamsCursorState;
```

---

### Finding 2

**File**: `apps/api/src/integrations/platforms/monday/providers/monday-client.provider.ts:114`  
**Category**: Type cast (`as T`)  
**Impact**: Medium

**Description**: `(await response.json()) as MondayGraphQLResponse<T>` casts the HTTP response body to the typed GraphQL response shape without validation. If the Monday API returns an unexpected shape (e.g., an error body from a CDN), the cast succeeds and subsequent `result.data` access will silently return `undefined`.

**Suggestion**: Parse the response through a Zod schema: `const parsed = mondayGraphQLResponseSchema.safeParse(await response.json())`. Define `mondayGraphQLResponseSchema` with `z.object({ data: z.unknown().optional(), errors: z.array(z.object({ message: z.string() })).optional() })`.

**Evidence**:

```typescript
// monday-client.provider.ts:114
const result = (await response.json()) as MondayGraphQLResponse<T>;
```

---

### Finding 3

**File**: `apps/api/src/integrations/platforms/monday/services/monday-sync.service.ts:329`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `integration.platformMetadata as { accountId?: string } | null` in `extractSyncConfig` casts without validation. If `platformMetadata` is an unexpected shape, `metadata?.accountId` will silently return `undefined`.

**Suggestion**: Use a Zod schema for the metadata shape: `z.object({ accountId: z.string().optional() }).nullable().safeParse(integration.platformMetadata)`.

**Evidence**:

```typescript
// monday-sync.service.ts:329
const metadata = integration.platformMetadata as { accountId?: string } | null;
```

---

### Finding 4

**File**: `apps/api/src/integrations/platforms/monday/services/monday-sync.service.ts:218`  
**Category**: Missing strict typing  
**Impact**: Medium

**Description**: `fetchThread` returns `Promise<unknown>`. This is the same pattern as `microsoft-onedrive-content-extraction.ts:150`. Callers of this method must cast or validate the returned value, pushing type unsafety into AI tool implementations.

**Suggestion**: Define a `MondayBoardFetchResult` interface and make the return type `Promise<MondayBoardFetchResult | null>`. The function already builds a well-defined object at line 312.

**Evidence**:

```typescript
// monday-sync.service.ts:218
async fetchThread(params: { userId: string; integrationId: string; threadId: string }): Promise<unknown> {
  // ...
  return {
    boardId: board.id,
    boardName: board.name,
    // ... well-defined fields
  };
}
```

---

### Finding 5

**File**: `apps/api/src/integrations/platforms/monday/services/monday-sync.service.ts:502`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `ctx.resumeState = nextResumeState as MondayCursorState` and `ctx.allItems = selection.selected.map((item) => itemCache.get(item.id)).filter(Boolean) as MondayItem[]` — both patterns recur here as in all other sync services.

**Suggestion**: Same fixes as for Jira/Linear/OneDrive/Outlook: use `satisfies MondayCursorState` on the object literal, and use typed filter predicate `(x): x is MondayItem => x !== undefined`.

**Evidence**:

```typescript
// monday-sync.service.ts:502
ctx.resumeState = nextResumeState as MondayCursorState;
// monday-sync.service.ts:511
ctx.allItems = selection.selected.map((item) => itemCache.get(item.id)).filter(Boolean) as MondayItem[];
```
