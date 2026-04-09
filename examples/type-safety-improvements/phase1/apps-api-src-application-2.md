# Audit: apps-api-src-application-2

**Files inspected**: 8
**Findings**: 5

## Summary

These files are capability-based use-cases for an AI agent layer, wrapping domain ports for integrations, memory, profile, project management, settings, todos, work items, and native search. The code is generally well-typed with `neverthrow` `Result` throughout, but several concrete issues were found: unsafe `as` casts that bypass TypeScript's type narrowing, `Record<string, unknown>` used where stricter mapped types would be safer, a loose `string` type where a constrained union exists on the port, and one redundant explicit type annotation on a `match` callback that is already fully narrowed.

## Findings

### Finding 1: `as` cast to avoid mapped-type index — should use satisfies or a typed helper

- **File**: `apps/api/src/application/agents/use-cases/manage-settings.use-case.ts:268`
- **Category**: type-cast
- **Impact**: medium
- **Description**: Inside `reset()`, the code builds an `updates` object by iterating over `fieldsToReset` (typed `(keyof Omit<UserSettings, "userId">)[]`). TypeScript correctly refuses the direct assignment `updates[field] = DEFAULT_SETTINGS[field]` because the index signature is not homomorphic here, so the author widened the type with `as Record<string, unknown>`. This silences the error but loses the type safety entirely: any value (including the wrong type) can now be assigned to any key without a compile-time error. The cast also bypasses the `in DEFAULT_SETTINGS` guard that was intended to provide safety.
- **Suggestion**: Use a typed helper that preserves the relationship between key and value:
  ```typescript
  function copyField<K extends keyof Omit<UserSettings, "userId">>(
    target: Partial<Omit<UserSettings, "userId">>,
    source: Omit<UserSettings, "userId">,
    key: K,
  ): void {
    target[key] = source[key];
  }
  // Then: copyField(updates, DEFAULT_SETTINGS, field);
  ```
  This removes the cast entirely and lets TypeScript verify the value type matches the key.
- **Evidence**:

```typescript
// manage-settings.use-case.ts:264-269
const updates: Partial<Omit<UserSettings, "userId">> = {};
for (const field of fieldsToReset) {
  if (field in DEFAULT_SETTINGS) {
    // Type assertion needed because TypeScript can't narrow the field type
    (updates as Record<string, unknown>)[field] = DEFAULT_SETTINGS[field];
  }
}
```

### Finding 2: `as NativeSearchUseCaseErrorType` cast — redundant and error-prone

- **File**: `apps/api/src/application/agents/use-cases/native-search.use-case.ts:224`
- **Category**: type-cast
- **Impact**: low
- **Description**: The ternary already produces either `"NO_INTEGRATIONS"` (a string literal assignable to `NativeSearchUseCaseErrorType`) or the string literal `"SEARCH_FAILED"`. Because both branches are string literals that are members of the union, TypeScript can infer the result as `NativeSearchUseCaseErrorType` without the cast. The cast as written does not add safety; if a typo were introduced in either branch the cast would silently hide it.
- **Suggestion**: Remove the cast and instead annotate the variable:
  ```typescript
  const errorType: NativeSearchUseCaseErrorType =
    error.code === "NO_INTEGRATIONS" ? "NO_INTEGRATIONS" : "SEARCH_FAILED";
  ```
  This lets TypeScript check both branches against the union without silencing errors.
- **Evidence**:

```typescript
// native-search.use-case.ts:223-225
const errorType =
  error.code === "NO_INTEGRATIONS" ? "NO_INTEGRATIONS" : ("SEARCH_FAILED" as NativeSearchUseCaseErrorType);
```

### Finding 3: `platform: string` in `resolveIntegrationId` — should be `ProjectManagementPlatform`

- **File**: `apps/api/src/application/agents/use-cases/manage-projects.use-case.ts:450`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: The private `resolveIntegrationId` helper accepts `platform: string`. The port (`project-management.port.ts`) imports and uses `ProjectManagementPlatform` (a string-literal union from `@slopweaver/contracts`: `"notion" | "asana" | "jira"`). All call-sites pass one of those three literals, but the loose `string` type on the helper allows any arbitrary string to be passed without a compile error. Similarly, `ManageProjectsError.platform` is typed `string` (line 117) even though it is always a `ProjectManagementPlatform` value.
- **Suggestion**: Import `ProjectManagementPlatform` from `@slopweaver/contracts` and tighten both the helper parameter and the error class field:

  ```typescript
  import { ProjectManagementPlatform } from "@slopweaver/contracts";

  private async resolveIntegrationId({
    userId,
    platform,
    providedIntegrationId,
  }: {
    userId: string;
    platform: ProjectManagementPlatform;
    providedIntegrationId?: string | undefined;
  }): Promise<Result<string, ManageProjectsError>>

  export class ManageProjectsError extends Error {
    constructor(
      public readonly code: ProjectManagementError["code"] | "NO_INTEGRATION",
      message: string,
      public readonly platform: ProjectManagementPlatform,
    ) { ... }
  }
  ```

- **Evidence**:

```typescript
// manage-projects.use-case.ts:113-122
export class ManageProjectsError extends Error {
  constructor(
    public readonly code: ProjectManagementError["code"] | "NO_INTEGRATION",
    message: string,
    public readonly platform: string,   // <-- should be ProjectManagementPlatform
  ) { ... }
}

// manage-projects.use-case.ts:444-452
private async resolveIntegrationId({
  userId,
  platform,
  providedIntegrationId,
}: {
  userId: string;
  platform: string;   // <-- should be ProjectManagementPlatform
  providedIntegrationId?: string | undefined;
}): Promise<Result<string, ManageProjectsError>>
```

### Finding 4: `Record<string, unknown>` for `filters` context object — weaker than a mapped type

- **File**: `apps/api/src/application/agents/use-cases/manage-todos.use-case.ts:93-97` and `299`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `ListTodosResultWithContext.query.filters` is typed `Record<string, unknown>`. The keys that are actually populated (`category`, `priority`, `status`, `completed`) are all known — they directly mirror the optional fields on `ListTodosQuery`. A stricter type would prevent silent key-name typos (e.g. `filters["catagory"]`) and allow callers to access values without casting. The same `Record<string, unknown>` is built inline in the `list()` method.
- **Suggestion**: Replace with an explicit mapped shape derived from the query fields:
  ```typescript
  filters: Partial<Pick<ListTodosQuery, "category" | "priority" | "status" | "completed">>;
  ```
  This is closed-world (no extra keys allowed), and each value retains its domain type (e.g. `status` stays `TodoStatus` rather than `unknown`).
- **Evidence**:

```typescript
// manage-todos.use-case.ts:91-97
export interface ListTodosResultWithContext extends ListTodosResult {
  query: {
    limit: number;
    offset: number;
    filters: Record<string, unknown>; // <-- all actual keys are known
  };
}

// manage-todos.use-case.ts:299
const filters: Record<string, unknown> = {};
```

### Finding 5: Redundant explicit type annotations on already-narrowed `match` callbacks

- **File**: `apps/api/src/application/agents/use-cases/manage-profile.use-case.ts:89` and `141`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: In `getContext()` and `getCommunicationStyle()`, the success and error callbacks passed to `.match()` have explicit parameter type annotations: `(context: ProfileContext)` and `(error: ProfileContextError)`. The `neverthrow` `Result<ProfileContext, ProfileContextError>` already constrains both callback parameter types, so the annotations are redundant noise. While not harmful on their own, they create a maintenance burden: if the `Result` type changes, the annotations must be updated in sync or they will silently drift. The wider codebase (other use-cases in this same batch) consistently omits these redundant annotations.
- **Suggestion**: Remove the inline type annotations from both callbacks and rely on inference:
  ```typescript
  return contextResult.match(
    (context) => { ... },   // inferred as ProfileContext
    (error) => { ... },     // inferred as ProfileContextError
  );
  ```
- **Evidence**:

```typescript
// manage-profile.use-case.ts:88-112
return contextResult.match(
  (context: ProfileContext) => {   // redundant — already inferred from Result<ProfileContext, ...>
    ...
  },
  (error: ProfileContextError) => {   // redundant — already inferred from Result<..., ProfileContextError>
    ...
  },
);
```
