# Audit: apps-api-src-application-35

**Files inspected**: 8
**Findings**: 7

## Summary

The most impactful issues are in `todo-extraction.service.ts`, which contains two identical copy-pasted patterns that unsafely access the `usage` property of the AI SDK's `generateText()` result — even though `GenerateTextResult.usage` is typed as `LanguageModelUsage` by the SDK and requires no cast at all. The `todo-proposals.repository.ts` file duplicates three type aliases (`TodoCategory`, `TodoPriority`, `TodoEffort`) that are already exported from `@slopweaver/contracts`. The `proposal-actions.repository.ts` has an inline `todoData` shape in `ensureAcceptedTodoAction` whose literal union types (`"bug" | "feature" | ...`) duplicate the contracts schemas. The `todo.service.ts` uses `Record<string, unknown>` to build a dynamic update payload where a stricter mapped type or Drizzle's `$inferInsert` partial would be safer. One redundant `as string` cast also appears in `todo-extraction.service.ts`.

---

## Findings

### Finding 1: Unnecessary type cast on AI SDK `generateText` result — duplicated twice

- **File**: `apps/api/src/application/todos/extraction/services/todo-extraction.service.ts:179-181` and `:486-488`
- **Category**: type-cast
- **Impact**: high
- **Description**: The code wraps `result` (which is `GenerateTextResult<...>`) in a runtime `typeof` guard and then casts it to `{ usage?: unknown }` before passing `usage` to `extractInputOutputTokensFromUsage`. The AI SDK's `GenerateTextResult` interface already declares `readonly usage: LanguageModelUsage` (confirmed in `ai/dist/index.d.ts` line 806). The guard and cast are therefore both redundant and misleading — they suggest `usage` might be absent, which is false.
- **Suggestion**: Remove the `typeof result === "object"` guard and the `as { usage?: unknown }` cast. Access `result.usage` directly.
- **Evidence**:

```typescript
// Current (lines 179-182) — unnecessary:
const usage =
  typeof result === "object" && result !== null && "usage" in result
    ? (result as { usage?: unknown }).usage
    : undefined;

// Correct:
const usage = result.usage;
```

---

### Finding 2: `cacheKey as string` cast is redundant

- **File**: `apps/api/src/application/todos/extraction/services/todo-extraction.service.ts:112`
- **Category**: type-cast
- **Impact**: low
- **Description**: `cacheEntry.key` is already typed as `string` on `CacheEntryHandle` (confirmed: `typed-cache.service.ts:326: readonly key: string`). The `as string` assertion is therefore a no-op and misleads readers into thinking a widening type is involved.
- **Suggestion**: Remove the cast.
- **Evidence**:

```typescript
// Current:
const cacheKey = cacheEntry.key as string;

// Correct:
const cacheKey = cacheEntry.key;
```

---

### Finding 3: `TodoCategory`, `TodoPriority`, `TodoEffort` type aliases duplicated from contracts

- **File**: `apps/api/src/application/todos/extraction/repositories/todo-proposals.repository.ts:24-26`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: Three local type aliases are defined in this file:
  ```typescript
  type TodoCategory = "bug" | "feature" | "meeting" | "review" | "communication" | "other";
  type TodoPriority = "urgent" | "high" | "medium" | "low";
  type TodoEffort = "5min" | "30min" | "2hr" | "1day" | "multi-day";
  ```
  These are identical to the exported `TodoCategory`, `TodoPriority`, and `TodoEffort` types in `@slopweaver/contracts` (derived from `todoCategorySchema`, `todoPrioritySchema`, `todoEffortSchema` in `packages/contracts/src/contracts/todos/schemas.ts`). If the contract enums ever change, these local copies will silently diverge.
- **Suggestion**: Import `TodoCategory`, `TodoPriority`, `TodoEffort` from `@slopweaver/contracts` and delete the local aliases.
- **Evidence**:

```typescript
// Current (todo-proposals.repository.ts:24-26):
type TodoCategory = "bug" | "feature" | "meeting" | "review" | "communication" | "other";
type TodoPriority = "urgent" | "high" | "medium" | "low";
type TodoEffort = "5min" | "30min" | "2hr" | "1day" | "multi-day";

// Contracts already export (packages/contracts/src/contracts/todos/types.ts):
export type TodoCategory = z.infer<typeof todoCategorySchema>;
export type TodoPriority = z.infer<typeof todoPrioritySchema>;
export type TodoEffort = z.infer<typeof todoEffortSchema>;
```

---

### Finding 4: Inline `todoData` shape in `ensureAcceptedTodoAction` duplicates contract literal unions

- **File**: `apps/api/src/application/todos/extraction/repositories/proposal-actions.repository.ts:157-168`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: The `todoData` parameter of `ensureAcceptedTodoAction` re-declares all five domain literal unions inline:
  ```typescript
  todoData: {
    category: "bug" | "feature" | "meeting" | "review" | "communication" | "other";
    effort: "5min" | "30min" | "2hr" | "1day" | "multi-day";
    priority: "urgent" | "high" | "medium" | "low";
    status: "pending";
    ...
  }
  ```
  The first four fields are already expressed by `TodoCategory`, `TodoEffort`, `TodoPriority`, and `TodoStatus` from `@slopweaver/contracts`. The `status: "pending"` could use `typeof TODO_STATUSES.PENDING` (already imported in `todo.service.ts`) or be expressed as `Extract<TodoStatus, "pending">`.
- **Suggestion**: Replace the four inline literal unions with the imported contract types, and use `Extract<TodoStatus, "pending">` for the status field.
- **Evidence**:

```typescript
// Current:
todoData: {
  category: "bug" | "feature" | "meeting" | "review" | "communication" | "other";
  content: string;
  effort: "5min" | "30min" | "2hr" | "1day" | "multi-day";
  priority: "urgent" | "high" | "medium" | "low";
  reasoning: string;
  status: "pending";
}

// Improved:
import type { TodoCategory, TodoEffort, TodoPriority, TodoStatus } from "@slopweaver/contracts";

todoData: {
  category: TodoCategory;
  content: string;
  effort: TodoEffort;
  priority: TodoPriority;
  reasoning: string;
  status: Extract<TodoStatus, "pending">;
}
```

---

### Finding 5: Three `as TodoCategory / as TodoEffort / as TodoPriority` casts required because `todoProposalsTable` columns are `text`

- **File**: `apps/api/src/application/todos/extraction/repositories/todo-proposals.repository.ts:217-220`
- **Category**: type-cast
- **Impact**: medium
- **Description**: The `listVisible` method casts three Drizzle `text` columns back to their domain enum types:
  ```typescript
  category: row.category as TodoCategory,
  estimatedEffort: row.estimatedEffort as TodoEffort,
  priority: row.priority as TodoPriority,
  ```
  These casts are required because `todoProposalsTable` defines `category`, `estimatedEffort`, and `priority` as plain `text` columns rather than typed enum columns. This means invalid values could be stored and silently coerced at read time with no runtime validation. This is a design-level issue: the table schema should either use `pgEnum` columns (generating proper DB enums) or validate the values through a Zod parse before the cast.
- **Suggestion**: Either (a) change the three columns to use `pgEnum` so Drizzle infers the correct type, eliminating the casts; or (b) add a Zod `.parse()` call after the DB read to validate the incoming strings. Option (a) is preferred as it closes the gap at schema level.
- **Evidence**:

```typescript
// Current (requires unsafe casts):
category: text("category").notNull(),           // todo-proposals.table.ts:18
estimatedEffort: text("estimated_effort").notNull(), // :21
priority: text("priority").notNull(),           // :23

// Better (would allow cast-free inference):
import { todoStatusEnum } from "@/shared/db/tables/enums";
// Define and use pgEnum columns for category, estimatedEffort, priority
```

---

### Finding 6: `Record<string, unknown>` for dynamic `updateData` accumulator in `updateTodo`

- **File**: `apps/api/src/application/todos/services/todo.service.ts:359`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: The `updateData` object that is spread into Drizzle's `.set()` call is typed as `Record<string, unknown>`. This means any key/value pair can be accumulated without compile-time validation against the actual table columns, and the subsequent spread into `.set({...updateData})` bypasses Drizzle's column-level type checking. A spurious key could silently be passed to `set()` with no error.
- **Suggestion**: Type `updateData` as `Partial<typeof todosTable.$inferInsert>` (excluding `id` and `userId`). This keeps the dynamic-construction pattern while narrowing the type to only valid table columns.
- **Evidence**:

```typescript
// Current:
const updateData: Record<string, unknown> = {};
for (const [key, value] of Object.entries(data)) {
  if (value !== undefined && key !== "dueDate" && key !== "snoozedUntil") {
    updateData[key] = value; // no column-name validation
  }
}

// Suggested:
type UpdateableColumns = Omit<typeof todosTable.$inferInsert, "id" | "userId" | "createdAt">;
const updateData: Partial<UpdateableColumns> = {};
```

---

### Finding 7: `OriginalTodo` interface in prompt utils duplicates `ExtractedTodo` from contracts

- **File**: `apps/api/src/application/todos/extraction/utils/todo-extraction-prompts.utils.ts:14-20`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: The `OriginalTodo` interface is defined locally:
  ```typescript
  export interface OriginalTodo {
    category: "bug" | "feature" | "meeting" | "review" | "communication" | "other";
    content: string;
    estimatedEffort: "5min" | "30min" | "2hr" | "1day" | "multi-day";
    priority: "urgent" | "high" | "medium" | "low";
    reasoning: string;
  }
  ```
  This is structurally identical to `ExtractedTodo` from `@slopweaver/contracts` (inferred from `extractedTodoSchema` in `packages/contracts/src/contracts/todos/schemas.ts:84-90`). Both have the same five fields with the same types. The `OriginalTodo` name is used in `clarifyExtraction`'s `answers: Record<string, string>` parameter and streamed output types, but could simply be replaced with `ExtractedTodo`.
- **Suggestion**: Delete `OriginalTodo` and replace all usages with `import type { ExtractedTodo } from "@slopweaver/contracts"`. This closes the risk of the two definitions diverging.
- **Evidence**:

```typescript
// Local (todo-extraction-prompts.utils.ts:14-20):
export interface OriginalTodo {
  category: "bug" | "feature" | "meeting" | "review" | "communication" | "other";
  content: string;
  estimatedEffort: "5min" | "30min" | "2hr" | "1day" | "multi-day";
  priority: "urgent" | "high" | "medium" | "low";
  reasoning: string;
}

// Contracts (packages/contracts/src/contracts/todos/schemas.ts:84-90):
export const extractedTodoSchema = z.object({
  category: todoCategorySchema, // "bug"|"feature"|"meeting"|"review"|"communication"|"other"
  content: z.string(),
  estimatedEffort: todoEffortSchema, // "5min"|"30min"|"2hr"|"1day"|"multi-day"
  priority: todoPrioritySchema, // "urgent"|"high"|"medium"|"low"
  reasoning: z.string(),
});
// export type ExtractedTodo = z.infer<typeof extractedTodoSchema>;
```
