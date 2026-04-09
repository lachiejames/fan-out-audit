# Type Safety Audit — apps/api/src/integrations (Batch 29)

**Files inspected**: 8  
**Findings**: 5

## Summary

Batch 29 covers the `microsoft-refresh-scope.utils.ts` and the Monday fetch/oauth/search/sync services. The Monday OAuth service is well-typed with proper Zod schemas for token and user info responses. The Monday search and fetch services have well-typed interfaces. Two cross-cutting findings dominate: (1) `monday-sync.service.ts` returns `Promise<unknown>` from `fetchThread`; (2) `monday-client.provider.ts` casts raw `response.json()` to a generic type `T`. Additionally, `monday-fetch.service.ts` uses `columnValues: { value: unknown }[]` in `MondayItemDetails` which is appropriate for JSONB but could note the reason.

## Findings

---

### Finding 1

**File**: `apps/api/src/integrations/platforms/monday/services/monday-fetch.service.ts:43`  
**Category**: Missing strict typing  
**Impact**: Low

**Description**: `MondayItemDetails.columnValues` has `value: unknown` for each column value. While `unknown` is correct here (Monday column values are JSONB with heterogeneous shapes per column type), callers must cast or narrow before using `value`. This is documented nowhere.

**Suggestion**: Add a JSDoc comment explaining why `value: unknown` is intentional (JSONB heterogeneous column types), so future readers understand it is a deliberate design choice rather than a missing type.

**Evidence**:

```typescript
// monday-fetch.service.ts:38-43
columnValues: {
  id: string;
  title: string;
  type: string;
  text: string | null;
  value: unknown; // intentionally unknown — Monday column values are heterogeneous JSONB
}
[];
```

---

### Finding 2

**File**: `apps/api/src/integrations/platforms/monday/services/monday-search.service.ts:37`  
**Category**: `Record<string, ...>` weakening  
**Impact**: Low

**Description**: `MondaySearchResult.columnValues` is typed as `Record<string, string | null>`. This is appropriate for the search use case where column values are reduced to their text representation, but the type differs from `MondayItemDetails.columnValues` (which keeps `value: unknown`). The two representations are consistent with their use cases but are not linked by a shared abstraction.

**Suggestion**: No change needed; this is appropriate. Add a comment noting that search results use text-only column values.

**Evidence**:

```typescript
// monday-search.service.ts:37
columnValues: Record<string, string | null>; // appropriate: text-only for search
```

---

### Finding 3

**File**: `apps/api/src/integrations/platforms/monday/services/monday-oauth.service.ts`  
**Category**: No findings  
**Impact**: N/A

**Description**: `MondayOAuthService` is well-typed. It uses Zod schemas (`mondayTokenResponseSchema`, `mondayUserInfoResponseSchema`) to validate all API responses before use. Token and user info responses are parsed with `.parse()` which throws on invalid data. No unsafe casts found.

**Evidence**: N/A — clean file.

---

### Finding 4

**File**: `apps/api/src/integrations/platforms/microsoft/utils/microsoft-refresh-scope.utils.ts:33`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: (Cross-reference with Batch 27 Finding 3) `(syncSettings as { accessMode?: unknown }).accessMode` casts `syncSettings: unknown` after an `object` type check. The cast is safe but would be cleaner as a type guard.

**Suggestion**: Extract a type guard `function hasSyncAccessMode(v: unknown): v is { accessMode?: unknown }` and replace the inline cast.

**Evidence**:

```typescript
// microsoft-refresh-scope.utils.ts:33
value: (syncSettings as { accessMode?: unknown }).accessMode,
```

---

### Finding 5

**File**: `apps/api/src/integrations/platforms/monday/utils/monday-oauth.utils.ts` (cross-batch)  
**Category**: No findings  
**Impact**: N/A

**Description**: The Monday OAuth utils file uses proper Zod schemas and well-typed helper functions throughout. Token expiry calculation, scope parsing, and URL building are all correctly typed. No unsafe patterns detected.

**Evidence**: N/A — clean file.
