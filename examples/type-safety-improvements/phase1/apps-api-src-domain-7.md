# Audit: apps-api-src-domain-7

**Files inspected**: 8
**Findings**: 7

## Summary

The eight domain port files are generally well-structured and follow hexagonal architecture principles. The main issues are: (1) `todos.port.ts` manually re-declares all the literal union types already exported from `@slopweaver/contracts` (TodoStatus, TodoPriority, TodoEffort, TodoCategory), causing four duplicate-type findings. (2) `TodoResult` uses widened `string` fields where the contracts already carry narrowed literal unions. (3) `ISettingsPort` and `ProfileContextPort` use positional parameters where the codebase mandates named-object params. (4) `IPromptCachePort.serializeProfileForCache` accepts a `Record<string, unknown>` where a more structured domain type would be safer.

---

## Findings

### Finding 1: Duplicate literal union types in todos.port.ts — TodoStatus, TodoPriority, TodoEffort, TodoCategory

- **File**: `apps/api/src/domain/ports/services/todos.port.ts:30–63`
- **Category**: duplicate-type
- **Impact**: high
- **Description**: `CreateTodoData`, `UpdateTodoData`, and `ListTodosQuery` inline the four literal unions — `"bug" | "feature" | "meeting" | "review" | "communication" | "other"`, `"urgent" | "high" | "medium" | "low"`, `"5min" | "30min" | "2hr" | "1day" | "multi-day"`, and the eight-value status union — as raw string literals. All four of these exact unions are already exported as `TodoCategory`, `TodoPriority`, `TodoEffort`, and `TodoStatus` from `@slopweaver/contracts`. Any future change to the canonical enums (e.g. adding a new status) will silently diverge from the port's inline copies.
- **Suggestion**: Import `TodoCategory`, `TodoPriority`, `TodoEffort`, and `TodoStatus` from `@slopweaver/contracts` and replace every inline literal union in `CreateTodoData`, `UpdateTodoData`, and `ListTodosQuery` with these imported types.
- **Evidence**:

```typescript
// todos.port.ts — duplicated inline unions
export interface CreateTodoData {
  category: "bug" | "feature" | "meeting" | "review" | "communication" | "other";
  effort: "5min" | "30min" | "2hr" | "1day" | "multi-day";
  priority: "urgent" | "high" | "medium" | "low";
  status: "proposed" | "accepted" | "pending" | "in_progress" | "completed" | "rejected" | "cancelled" | "snoozed";
  // ...
}

// @slopweaver/contracts — canonical source of truth
export type TodoCategory = z.infer<typeof todoCategorySchema>; // same union
export type TodoPriority = z.infer<typeof todoPrioritySchema>; // same union
export type TodoEffort = z.infer<typeof todoEffortSchema>; // same union
export type TodoStatus = z.infer<typeof todoStatusSchema>; // same union
```

---

### Finding 2: TodoResult uses widened `string` fields instead of contract literal types

- **File**: `apps/api/src/domain/ports/services/todos.port.ts:68–79`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `TodoResult.status`, `priority`, `category`, and `effort` are all typed as plain `string`. Callers that receive a `TodoResult` therefore lose the literal-union narrowing and cannot exhaustively switch on these fields without a cast. The contract's `Todo` type (from `@slopweaver/contracts`) already carries the narrowed unions for all four fields.
- **Suggestion**: Either reuse the contract's `Todo` type directly as the return type, or tighten `TodoResult` field types to `TodoStatus`, `TodoPriority`, `TodoCategory`, and `TodoEffort`.
- **Evidence**:

```typescript
export interface TodoResult {
  id: string;
  content: string;
  status: string; // should be TodoStatus
  priority: string; // should be TodoPriority
  category: string; // should be TodoCategory
  effort: string; // should be TodoEffort
  dueDate: string | null;
  completedAt: string | null;
  createdAt: string;
  updatedAt: string;
}
```

---

### Finding 3: ISettingsPort methods use positional parameters — violates named-object-params rule

- **File**: `apps/api/src/domain/ports/services/settings.port.ts:41,48`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: Both abstract methods use bare positional parameters (`userId: string` and `updates: Partial<UserSettings>`). The codebase mandates named object params for all functions with one or more parameters (see `typescript-patterns.md`). The current signature allows callers to accidentally swap arguments without a compile-time error.
- **Suggestion**: Wrap both signatures in a named params object:

```typescript
abstract getSettings(input: { userId: string }): Promise<UserSettings>;
abstract updateSettings(input: { userId: string; updates: Partial<UserSettings> }): Promise<void>;
```

- **Evidence**:

```typescript
// settings.port.ts — positional params
abstract getSettings(userId: string): Promise<UserSettings>;
abstract updateSettings(userId: string, updates: Partial<UserSettings>): Promise<void>;
```

---

### Finding 4: ProfileContextPort methods use positional parameters — violates named-object-params rule

- **File**: `apps/api/src/domain/ports/services/profile-context.port.ts:49,58`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `getProfileContext(userId: string)` and `refreshProfile(userId: string)` both take a bare positional `userId` parameter. While a single `string` param may seem harmless, the codebase rule applies to all functions with 1+ params. This is inconsistent with the majority of ports in the same directory (e.g. `ITokenCapPort.getUsageStats`, `ITriageClassifierPort.classifyItems`) which correctly use `input: { userId: string }`.
- **Suggestion**: Refactor to named-object params:

```typescript
abstract getProfileContext(input: { userId: string }): Promise<Result<ProfileContext, ProfileContextError>>;
abstract refreshProfile(input: { userId: string }): Promise<Result<void, ProfileContextError>>;
```

- **Evidence**:

```typescript
// profile-context.port.ts — positional params
abstract getProfileContext(userId: string): Promise<Result<ProfileContext, ProfileContextError>>;
abstract refreshProfile(userId: string): Promise<Result<void, ProfileContextError>>;
```

---

### Finding 5: ITokenCapPort.canProceedWithAICall uses positional parameter — violates named-object-params rule

- **File**: `apps/api/src/domain/ports/services/token-cap.port.ts:38`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `canProceedWithAICall(userId: string)` uses a bare positional string, unlike the sibling method `getUsageStats(input: { userId: string })` which correctly uses a named-object param. The inconsistency is minor but violates the codebase rule.
- **Suggestion**: Change to `canProceedWithAICall(input: { userId: string }): Promise<Result<void, TokenCapError>>`.
- **Evidence**:

```typescript
// token-cap.port.ts
abstract canProceedWithAICall(userId: string): Promise<Result<void, TokenCapError>>;  // positional
abstract getUsageStats(input: { userId: string }): Promise<TokenUsageStats>;           // named ✓
```

---

### Finding 6: IPromptCachePort.serializeProfileForCache accepts Record<string, unknown> — overly wide

- **File**: `apps/api/src/domain/ports/services/prompt-cache.port.ts:69`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `serializeProfileForCache(profile: Record<string, unknown>)` accepts any arbitrary object. The callers in practice always pass a `ProfileContext`-derived object (or similar structured domain type). Using `Record<string, unknown>` suppresses type-checking at the call site and makes the port contract opaque. There is no safety net if a caller accidentally passes an unrelated object.
- **Suggestion**: If the intent is to serialize any JSON-serializable object (e.g. the compiled profile data blob), introduce a named `SerializableProfile` type or at minimum document the expected shape. If only `ProfileContext` subtypes are ever passed, tighten the parameter to that type.
- **Evidence**:

```typescript
abstract serializeProfileForCache(profile: Record<string, unknown>): string;
```

---

### Finding 7: IPromptCachePort model literal union duplicated across multiple methods

- **File**: `apps/api/src/domain/ports/services/prompt-cache.port.ts:94,99,108`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: The literal union `"sonnet" | "opus" | "haiku"` appears three times inline across `estimateSavings`, `estimateSavingsForVolume`, and `meetsCacheThreshold`. It is not extracted to a named type, so any addition of a new model tier (e.g. `"haiku-3"`) requires updating three separate call sites.
- **Suggestion**: Extract to a named type: `type CacheableModel = "sonnet" | "opus" | "haiku"` and use it in all three method signatures.
- **Evidence**:

```typescript
abstract estimateSavings(input: { cachedTokens: number; model?: "sonnet" | "opus" | "haiku" }): number;
abstract estimateSavingsForVolume(input: { ...; model?: "sonnet" | "opus" | "haiku" }): number;
abstract meetsCacheThreshold(input: { content: string; model?: "sonnet" | "opus" | "haiku" }): boolean;
```
