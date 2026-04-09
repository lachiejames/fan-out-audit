# Audit: apps-api-src-seed-data-1

**Findings count**: 2
**Summary**: Two issues in seed-data fixtures. `embeddings.ts` is an `AUTO-GENERATED` file (re-exports exempted by codebase rules) and `contents.ts` is too large to read in full but its header confirms `satisfies NewContent[]`. The main finding is in `sync-checkpoints.ts` where a local `CursorState = Record<string, unknown>` type alias is used for `satisfies` typing but the actual cursor shapes vary per platform and could be typed more precisely. `knowledge-items.ts` inlines embedding arrays directly alongside item data, duplicating the embedding storage pattern from `embeddings.ts`.

---

## Finding 1

**File**: `apps/api/src/seed-data/sync-checkpoints.ts`
**Line**: 11
**Category**: Weak `Record<string, …>` type
**Impact**: Low — the `CursorState = Record<string, unknown>` alias is used only for the `satisfies` expression on `cursorState` fields within this fixture file. It accepts any object, so a typo in a cursor key would not be caught at compile time.

**Description**: Each checkpoint's `cursorState` has a distinct shape per platform: Gmail uses `{ historyId, lastMessageId, lastMessageTimestamp }`, Slack uses `{ lastMessageTs, oldestTs }`, Linear uses `{ lastUpdatedAt }`. These shapes are known. A per-platform type or a discriminated union would make the cursor states self-documenting and prevent key typos.

**Suggestion**: Define named cursor state interfaces (e.g. `GmailCursorState`, `SlackCursorState`, `LinearCursorState`) in a shared sync-checkpoints types file and use them instead of the local alias. If the DB column is typed `unknown` / `jsonb`, the `satisfies` approach still works but provides richer checking.

**Evidence**:

```typescript
type CursorState = Record<string, unknown>;
// ...
cursorState: {
  historyId: "5000001",
  lastMessageId: "demo-gmail-015",
  lastMessageTimestamp: "1737964800000",
} satisfies CursorState,
```

---

## Finding 2

**File**: `apps/api/src/seed-data/knowledge-items.ts`
**Lines**: 13 onward
**Category**: Duplicate type definition (structural)
**Impact**: Low — the file declares a local `knowledgeEmbeddings: Record<string, number[]>` const that stores pre-computed embedding vectors inline alongside the knowledge item data. The `embeddings.ts` file in the same directory serves this purpose for content embeddings. Having two separate embedding storage patterns in the same seed-data layer is inconsistent.

**Description**: Knowledge item embeddings are stored in a local `Record<string, number[]>` map within `knowledge-items.ts` rather than following the pattern in `embeddings.ts` (which is AUTO-GENERATED and stores content embeddings by ID). This inconsistency means knowledge item embeddings are not covered by `pnpm cli generate-demo-embeddings`.

**Suggestion**: Either move knowledge item embeddings to a separate `knowledge-embeddings.ts` with an `AUTO-GENERATED` header (consistent with `embeddings.ts`), or document why the inline approach is intentional. No type-safety fix needed, but the `Record<string, number[]>` type should at minimum be `Record<string, readonly number[]>` to prevent accidental mutation.

**Evidence**:

```typescript
const knowledgeEmbeddings: Record<string, number[]> = {
  "d3m0-know-0000-0000-000000000001": [ ... ],
  // ...
};
```
