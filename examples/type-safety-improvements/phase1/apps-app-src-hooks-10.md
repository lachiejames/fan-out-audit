# Type-Safety Audit: apps-app-src-hooks-10

Files audited:

- `apps/app/src/hooks/useTodoMutations.ts`
- `apps/app/src/hooks/useTodosData.ts`
- `apps/app/src/hooks/useTodoUpdateMutation.ts`
- `apps/app/src/hooks/useTriage.ts`
- `apps/app/src/hooks/useTriageKeyboardShortcuts.ts`
- `apps/app/src/hooks/useTriageStats.ts`
- `apps/app/src/hooks/useTriageTrigger.ts`
- `apps/app/src/hooks/useUpdateIntegration.ts`

---

## Finding 1

**File:** `apps/app/src/hooks/useTodoMutations.ts` (lines 62, 143, 402)

**Category:** `any` or unsafe `unknown` in production code

**Impact:** Medium

**Description:**
Three result interfaces — `UseCompleteTodoResult`, `UseAcceptProposalResult`, and `UseCreateTodoResult` — declare `mutateAsync` with a `Promise<unknown>` return type. In all three cases the underlying mutation function returns a typed value (e.g., the created todo's body from the contract), but the public interface discards that type.

**Suggestion:**
Derive the return type from the actual mutation function. For `useCreateTodo`, `mutateAsync` should return `Promise<TodoResponse>` (or whatever the `tsr.todos.createTodo` 201 body type is). This allows callers to access `data.id` after creation without casting.

**Evidence:**

```typescript
mutateAsync: (todoId: string) => Promise<unknown>        // UseCompleteTodoResult
mutateAsync: (params: {...}) => Promise<unknown>          // UseAcceptProposalResult
mutateAsync: (content: string) => Promise<unknown>       // UseCreateTodoResult
```

---

## Finding 2

**File:** `apps/app/src/hooks/useTodoUpdateMutation.ts` (line 31)

**Category:** `any` or unsafe `unknown` in production code

**Impact:** Low

**Description:**
`UseUpdateTodoResult` declares `mutateAsync` as returning `Promise<unknown>`. The underlying `mutationFn` returns `response.body` (the typed 200 body from the update contract) or `null` (when no fields changed), so the actual type is `TodoResponse | null`.

**Suggestion:**
Type `mutateAsync` as `Promise<TodoResponse | null>` to propagate the known result type to callers.

**Evidence:**

```typescript
mutateAsync: (params: {...}) => Promise<unknown>  // UseUpdateTodoResult
```

---

## Finding 3

**File:** `apps/app/src/hooks/useTodosData.ts` (lines 18–49)

**Category:** Custom types duplicating SDK/contract types

**Impact:** High

**Description:**
Four local type aliases — `TaskCategory`, `TaskEffort`, `TaskPriority`, `TaskStatus` — and the `Task` interface are defined in this file. These map 1:1 to the todo response schema from `@slopweaver/contracts`. The mapping is performed in `transformTodo` in `todo-mapping.utils.ts`, which means the frontend uses these local types while the backend uses the contract types — and the two can silently diverge.

The `Task` interface specifically mirrors most fields of the contract's `TodoResponse` type but uses different value literals (e.g. `"urgent" | "important" | "normal"` for priority vs. the contract's `"urgent" | "high" | "medium" | "low"`). This is an intentional adaptation for the legacy UI, but it means there is no shared type contract between the API layer and the view layer.

**Suggestion:**
This is a systemic issue. The recommended path is to deprecate the local `Task` type and migrate components to use the contract's `TodoResponse` type directly, adapting display logic (e.g., priority label mapping) in isolated utility functions rather than in a separate interface. In the interim, document why the mismatch exists so the divergence is intentional and tracked.

**Evidence:**

```typescript
export type TaskCategory = "reply" | "meeting" | "review" | "task" | "followup";
export type TaskEffort = "tiny" | "small" | "medium" | "large";
export type TaskPriority = "urgent" | "important" | "normal";
export type TaskStatus = "todo" | "in_progress" | "done";
export interface Task { ... }
```

---

## Finding 4

**File:** `apps/app/src/hooks/useTriage.ts` (line 65)

**Category:** `any` or unsafe `unknown` in production code

**Impact:** Low

**Description:**
`_err: unknown` in the `onError` callback of `useProcessItems` is the standard TanStack Query `onError` type. This is acceptable. However, the usage of `ErrorResponse` type cast on line in `useTriageTrigger.ts` (not this file) is noted for completeness.

No actionable finding for `useTriage.ts` itself — the file correctly imports contract types (`TriageAction`, `TriageItemStatus`, `TriageSessionWithItems`) and uses them throughout.

---

## Finding 5

**File:** `apps/app/src/hooks/useTriageTrigger.ts` (line 65)

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Low

**Description:**
`result.body as ErrorResponse` is used to extract an error message from a non-200/non-204 response. `ErrorResponse` is imported from `@slopweaver/contracts` — this is a better pattern than an inline object cast, but `extractErrorMessage` already accepts and handles `ErrorResponse` bodies safely, so the cast can still be eliminated.

**Suggestion:**
Use `extractErrorMessage({ body: result.body, fallback: "..." })` directly without the intermediate `as ErrorResponse` cast.

**Evidence:**

```typescript
extractErrorMessage({ body: result.body as ErrorResponse, fallback: "..." });
```

---

## Finding 6

**File:** `apps/app/src/hooks/useUpdateIntegration.ts` (lines 29–33, 67)

**Category:** `Record<string, ...>` where stricter types would be safer

**Impact:** Medium

**Description:**
`UpdateIntegrationParams.syncSettings` is typed as `Record<string, unknown>`. The sync settings body passed to the API has a known schema in the contracts package — universal settings include `messageLimit`, `syncProfile`, etc. Using `Record<string, unknown>` loses all type guidance for callers.

Additionally, line 67 uses `response.body as { message?: string | string[] }` to extract an error message from a non-200 response — the same pattern addressed elsewhere in this audit.

**Suggestion:**

- Replace `syncSettings?: Record<string, unknown>` with the typed `IntegrationSyncSettings` (or equivalent) from `@slopweaver/contracts`.
- Replace the cast on line 67 with `extractErrorMessage` from `@slopweaver/contracts`.

**Evidence:**

```typescript
syncSettings?: Record<string, unknown>                    // UpdateIntegrationParams
response.body as { message?: string | string[] }          // line 67
```

---

## Finding 7

**File:** `apps/app/src/hooks/useTriageKeyboardShortcuts.ts`

No type-safety issues found. The file defines a clean local `TriageAction` type that mirrors the contract type (acceptable since it is only used for keyboard shortcut mapping), and all state is properly typed.

---

## Finding 8

**File:** `apps/app/src/hooks/useTriageStats.ts`

No type-safety issues found. The file correctly imports all types from `@slopweaver/contracts` (`TriageDailyActivity`, `TriageLifetimeStats`, `TriageWeeklyTrend`), uses `select: (d) => d.body` consistently, and has no unsafe casts.
