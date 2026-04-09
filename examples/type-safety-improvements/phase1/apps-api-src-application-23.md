# Audit: apps-api-src-application-23

**Files inspected**: 8
**Findings**: 4

## Summary

These files cover the memory filesystem, network graph aggregation, notifications CRUD, and several error-type modules. The error files are well-typed discriminated unions with no issues. The main findings are: a hand-rolled `MemoryLearning` interface that duplicates the Drizzle-inferred `Knowledge` type, a hand-rolled `NetworkNode` interface that duplicates the Zod-inferred contract type, an unsafe `sql<Date>` cast in the network query, and an implicit `unknown` catch binding in `memory.service.ts` that is passed directly to a domain error constructor without any narrowing.

## Findings

### Finding 1: `MemoryLearning` duplicates the Drizzle-inferred `Knowledge` type

- **File**: `apps/api/src/application/memory/services/memory.service.ts:22`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `MemoryLearning` is a hand-written interface whose fields (`id`, `userId`, `category`, `content`, `confidence`, `sourceContentIds`, `createdAt`, `timesUsed`, `lastUsedAt`) exactly mirror the columns selected from `knowledgeItemsTable`. The table already exports `Knowledge = typeof knowledgeItemsTable.$inferSelect`. The only difference is that `createdAt` and `lastUsedAt` are typed as `string`/`string | null` here instead of `Date`/`Date | null` — which is the serialised form after `.toISOString()`. This means the interface represents the _serialised_ shape, but the contract already defines that via `ServerInferResponses`. There is no type-level guarantee that the hand-written interface stays in sync with the table schema; a column rename or type change in the table will silently diverge.
- **Suggestion**: Derive the serialised type from the Drizzle inferred type instead of writing it by hand. Use `Omit<Knowledge, 'createdAt' | 'lastUsedAt'> & { createdAt: string; lastUsedAt: string | null }`, or better, drop `MemoryLearning` altogether and let the `ListLearningsResult` item type be `ServerInferResponses<typeof contract.memory.listLearnings, 200>["body"]["items"][number]` so it is always contract-aligned.
- **Evidence**:

```typescript
// memory.service.ts:22-32
interface MemoryLearning {
  id: string;
  userId: string;
  category: string;
  content: string;
  confidence: number;
  sourceContentIds: string[];
  createdAt: string;
  timesUsed: number;
  lastUsedAt: string | null;
}

// shared/db/tables/knowledge-items.table.ts:113
export type Knowledge = typeof knowledgeItemsTable.$inferSelect;
// Knowledge already has: id, userId, category, content, confidence,
// sourceContentIds, createdAt (Date), timesUsed, lastUsedAt (Date | null)
```

---

### Finding 2: `NetworkNode` duplicates the contract's `networkNodeResponseSchema` type

- **File**: `apps/api/src/application/network/services/network.service.ts:19`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `NetworkNode` is defined locally with fields (`email`, `frequency`, `id`, `lastContact`, `messageCount`, `name`, `platform`, `sentiment`) that are structurally identical to the Zod schema `networkNodeResponseSchema` in `@slopweaver/contracts` (which defines the exact same fields with the same nullability). If the contract schema ever gains or renames a field, the local interface will silently diverge. The service builds `NetworkNode[]` values and then returns them as the contract response type without any runtime re-validation, so any drift becomes an invisible mismatch.
- **Suggestion**: Replace the local `NetworkNode` interface with `z.infer<typeof networkNodeResponseSchema>` imported from `@slopweaver/contracts`, or use `ServerInferResponses<typeof contract.network.getNetworkGraph, 200>["body"]["nodes"][number]`. Similarly, replace `NetworkGraphResult` with the inferred response body type. This makes the compiler enforce alignment with the contract automatically.
- **Evidence**:

```typescript
// network.service.ts:19-32
interface NetworkNode {
  email: string | null;
  frequency: number;
  id: string;
  lastContact: Date;
  messageCount: number;
  name: string;
  platform: string;
  sentiment: "positive" | "neutral" | "mixed" | null;
}

// packages/contracts/src/contracts/network/schemas.ts:17-26
const baseNetworkNodeSchema = z.object({
  email: z.string().nullable(),
  frequency: z.number().int().min(0).max(100),
  id: z.string(),
  lastContact: isoDateTimeStringSchema,
  messageCount: z.number().int().min(0),
  name: z.string(),
  platform: networkPlatformSchema,
  sentiment: networkSentimentSchema.nullable(),
});
```

---

### Finding 3: Unsafe `sql<Date>` cast in network query

- **File**: `apps/api/src/application/network/services/network.service.ts:59`
- **Category**: type-cast
- **Impact**: low
- **Description**: `sql<Date>\`max(${contentsTable.timestamp})\`.as("last_contact")`uses a generic type parameter to assert the Drizzle SQL fragment returns a`Date`. Drizzle does not validate this — the actual runtime value from the driver is whatever the pg driver returns for a `max()`aggregate on a`timestamptz`column, which is typically a`Date`object when the column is defined with`mode: 'date'`. However, the assertion silently breaks if the column mode or driver behaviour ever changes, and there is no narrowing or runtime check. The companion `sql<number>` cast on line 60 has the same pattern.
- **Suggestion**: These casts are a practical necessity with Drizzle's `sql` template literal and are widely used in the codebase. The lowest-risk improvement is to add an inline comment (`// pg driver returns Date for timestamptz max()`) to document the assumption. If strict safety is required, add a runtime guard after the query: `if (!(contact.lastContact instanceof Date)) throw new Error(...)`.
- **Evidence**:

```typescript
// network.service.ts:59-60
lastContact: sql<Date>`max(${contentsTable.timestamp})`.as("last_contact"),
messageCount: sql<number>`cast(count(*) as integer)`.as("message_count"),
```

---

### Finding 4: Implicit `unknown` catch binding passed directly to domain error constructor without narrowing

- **File**: `apps/api/src/application/memory/services/memory.service.ts:183`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: In `clearSession`, the `catch` block captures `error` without a type annotation (implicit `unknown` in TypeScript's `useUnknownInCatchVariables` strict mode). The raw `error` value is then passed directly into `MemoryErrors.database(...)` as the `cause` argument. While `cause` is typed as `unknown` in the error constructor so this does not cause a compile error, the `error` value is never narrowed or inspected before use. This is inconsistent with the rest of the file (`memory-readonly-files.service.ts`) where all catch blocks explicitly annotate `error: unknown`. The lack of annotation here also means if `useUnknownInCatchVariables` is ever disabled, the type silently widens to `any`.
- **Suggestion**: Add the explicit type annotation `catch (error: unknown)` to match the pattern used throughout `memory-readonly-files.service.ts` and the rest of the codebase.
- **Evidence**:

```typescript
// memory.service.ts:183-190  (note: no `: unknown` annotation)
    } catch (error) {
      this.logger.error({
        error,
        msg: "MemoryService.clearSession failed",
        userId,
      });
      return err(MemoryErrors.database("Failed to clear session state", error));
    }

// vs. consistent pattern everywhere else, e.g. memory-readonly-files.service.ts:117
    } catch (error: unknown) {
```
