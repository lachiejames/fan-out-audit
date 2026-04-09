# Audit: apps-api-src-shared-4

**Files inspected**: 8
**Findings**: 4

## Summary

The Drizzle table definitions in this slice are generally well-typed with `$type<>` annotations on jsonb columns. However, `action-transactions.table.ts` uses `$type<Record<string, unknown>>` on its metadata column (a weak type), `anthropic-files.table.ts` uses a bare `text` column for `purpose` where an enum or union would be more precise, and `behavioral-fingerprint.table.ts` has jsonb columns typed with local `Record<K, V>` aliases that are sound but underdocumented.

## Findings

### Finding 1: `metadata` column typed as `Record<string, unknown>` in action-transactions

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/action-transactions.table.ts:26`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: The `metadata` column is typed as `Record<string, unknown>`, which is only marginally better than untyped. The comment says it holds "action type, model used, Lemon Squeezy order ID" — all of which are known fields. Queries on this table that read metadata will need to cast every access.
- **Suggestion**: Define an `ActionTransactionMetadata` interface with known optional fields (`actionType?: string`, `modelUsed?: string`, `lemonSqueezyOrderId?: string`, etc.) and use `.$type<ActionTransactionMetadata>()`.
- **Evidence**:

```typescript
metadata: jsonb("metadata").$type<Record<string, unknown>>(),
```

### Finding 2: `purpose` column in `anthropic-files.table.ts` is untyped `text` when a union is known

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/anthropic-files.table.ts:18`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The `purpose` column has a comment listing valid values: `'core-profile', 'knowledge', 'patterns', 'context-pack'`. The column type is plain `text` with no `.$type<>` constraint. This means any string is accepted at insert time and returned without narrowing at select time.
- **Suggestion**: Add `.$type<"core-profile" | "knowledge" | "patterns" | "context-pack">()` to the column definition. This brings the TS type in line with the DB-level semantics described in the comment.
- **Evidence**:

```typescript
purpose: text("purpose").notNull(), // 'core-profile', 'knowledge', 'patterns', 'context-pack'
```

### Finding 3: `formalityByRecipient`, `responseSpeedByRecipient`, `toneByChannel` in behavioral-fingerprint use only `Record` aliases

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/behavioral-fingerprint.table.ts:13-19`
- **Category**: record-weakening
- **Impact**: low
- **Description**: Three type aliases are defined for the jsonb columns: `FormalityByRecipient = Record<string, number>`, `ResponseSpeedByRecipient = Record<string, number>`, and `ToneByChannel = Record<string, "formal" | "casual" | "balanced">`. The key type is `string` (entity ID), which is reasonable. The value types are appropriately typed for `ToneByChannel`, but `FormalityByRecipient` and `ResponseSpeedByRecipient` have the same structure (`Record<string, number>`) and are structurally identical — they are interchangeable at the type level.
- **Suggestion**: These are acceptable as-is but could use a branded type (`type EntityId = string & { readonly __brand: "EntityId" }`) for the key to distinguish them from generic `Record<string, number>`. Lower priority if entity IDs are never validated separately.
- **Evidence**:

```typescript
export type FormalityByRecipient = Record<string, number>;
export type ResponseSpeedByRecipient = Record<string, number>;
```

### Finding 4: `metadata` column in `chat-messages.table.ts` — no `metadata` column, but `uiParts` is typed via contracts

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/chat-messages.table.ts:40`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The `uiParts` column correctly uses `.$type<UiPart[]>()` imported from `@slopweaver/contracts` — this is the gold standard pattern. No issues found in this file. Documented here to confirm the positive pattern was observed.
- **Suggestion**: No action needed. This finding is informational only.
- **Evidence**:

```typescript
uiParts: jsonb("ui_parts").$type<UiPart[]>().notNull().default(sql`'[]'::jsonb`),
```
