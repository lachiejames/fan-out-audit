# Audit: apps-app-src-utils-3

**Files inspected**: 6
**Findings**: 7

## Summary

This slice covers queue audit formatting, queue trust display, the search-result transformer (covered in utils-2), todo mapping/query utilities, and tool-label utilities. The dominant issues are: `details: Record<string, unknown>` in queue audit formatting with a cascade of `as string` bracket casts for every field extraction; `AI_MODEL_INFO[modelId as AIModelId]` in queue-trust display casting without a type guard; a large inline `transformTodo` parameter type in todo-mapping that should use the imported contract type directly; `TodoQueryParams.status` duplicating the contracts status literal union; and a local `ToolCallStatus` type in `tool-label.utils.ts` that duplicates the same type in `@/types/assistant`. Additionally, `formatPlatformName` in `queue-audit-formatting.utils.ts` reimplements platform display-name logic that is already available from `@slopweaver/contracts`.

---

## Findings

### Finding 1: `details: Record<string, unknown>` in queue audit formatting with cascading `as string` casts

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/queue-audit-formatting.utils.ts:35-80`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: `formatAuditDetails` accepts `details: Record<string, unknown>` and extracts every field via bracket notation with `as string` (or `as number`) casts: `details["platform"] as string`, `details["messageCount"] as number`, `details["errorCode"] as string`, etc. Each cast silently returns `undefined` if the field is missing or has the wrong runtime type, with no validation. The queue audit action types are known (`"sync"`, `"triage"`, `"ai_reply"`, etc.), and each has a distinct well-known details shape.
- **Suggestion**: Define a discriminated union of audit detail types keyed by action type (e.g. `SyncAuditDetails`, `TriageAuditDetails`) and pass the discriminated union as the `details` parameter. Use a type guard (`isSyncAuditDetails`) before accessing action-specific fields. This eliminates all `as` casts and surfaces missing fields at compile time.
- **Evidence**: `const platform = details["platform"] as string;`, `const messageCount = details["messageCount"] as number;`, and multiple similar patterns throughout `formatAuditDetails`.

### Finding 2: `formatPlatformName` duplicates contract platform display name logic

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/queue-audit-formatting.utils.ts:10-28`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `formatPlatformName` contains a `switch` over platform ID strings (e.g. `"google-gmail"` → `"Gmail"`, `"microsoft-outlook"` → `"Outlook"`) returning display names. `@slopweaver/contracts` exports the `PLATFORMS` registry which has a `display.displayName` field for every platform. This function duplicates that mapping — if a new platform is added to contracts, the `formatPlatformName` switch will silently fall through to the `default` case, returning the raw ID instead of the display name.
- **Suggestion**: Replace the switch with `PLATFORMS[platformId as PlatformId]?.display?.displayName ?? platformId`. Add an `isPlatformId` guard before the lookup to avoid the unsafe cast, or use `PLATFORMS[platformId as keyof typeof PLATFORMS]?.display.displayName`.
- **Evidence**: `function formatPlatformName(platform: string): string { switch (platform) { case "google-gmail": return "Gmail"; ... } }`

### Finding 3: `AI_MODEL_INFO[modelId as AIModelId]` cast without validation in queue trust display

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/queue-trust-display.utils.ts:22`
- **Category**: type-cast
- **Impact**: low
- **Description**: `AI_MODEL_INFO[modelId as AIModelId]` casts an arbitrary `string` parameter to `AIModelId` for the record lookup. The function accepts `modelId: string` and the cast pretends the input is always a valid `AIModelId`. If an unknown model ID arrives from the API, the lookup returns `undefined`, but the return type does not reflect this — the caller receives `undefined` while TypeScript believes it received a valid `AIModelInfo` object.
- **Suggestion**: Add an `isAIModelId(modelId)` type predicate before the lookup and update the return type to `AIModelInfo | undefined` (or provide a fallback). Alternatively, change the parameter type to `AIModelId` and have callers validate before calling.
- **Evidence**: `return AI_MODEL_INFO[modelId as AIModelId];`

### Finding 4: `transformTodo` uses a large inline object type instead of the imported contract type

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/todo-mapping.utils.ts:18-60`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `transformTodo` accepts an `apiTodo` parameter typed as a large inline object literal with 20+ fields. The `@slopweaver/contracts` package exports a `TodoResponse` (or equivalent) type that describes the exact same shape returned by the API. The inline type must be manually kept in sync with the contract — if the API adds or renames a field, the inline type silently diverges.
- **Suggestion**: Replace the inline parameter type with the imported contract type: `function transformTodo({ apiTodo }: { apiTodo: TodoResponse }): ClientTodo`. This ensures the compiler flags mismatches as soon as the contract changes.
- **Evidence**: `function transformTodo(apiTodo: { id: string; userId: string; content: string; status: "pending" | "done" | ...; priority: "urgent" | ...; sourceMetadata: Record<string, unknown>; ... })` — large inline type instead of `TodoResponse` from contracts.

### Finding 5: `sourceMetadata: Record<string, unknown>` with bracket `as string` casts in `transformTodo`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/todo-mapping.utils.ts:55-75`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: `transformTodo` reads `apiTodo.sourceMetadata["platform"] as string`, `apiTodo.sourceMetadata["integrationId"] as string`, `apiTodo.sourceMetadata["externalId"] as string`, etc. via bracket notation casts. The source metadata for todos has a known structure (it contains the platform, integration ID, external ID, and URL). Typing `sourceMetadata` as `Record<string, unknown>` forces every consumer to re-cast fields that are already known.
- **Suggestion**: Define a `TodoSourceMetadata` interface (or import it from contracts if it exists) with the known fields typed precisely: `platform: PlatformId; integrationId: string; externalId: string; url?: string`. Use it as the type for `sourceMetadata` in `transformTodo` and wherever else todo metadata is consumed.
- **Evidence**: `apiTodo.sourceMetadata["platform"] as string`, `apiTodo.sourceMetadata["integrationId"] as string` and similar patterns in `transformTodo`.

### Finding 6: `TodoQueryParams.status` duplicates the contracts status literal union

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/todo-query.utils.ts:12`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: `TodoQueryParams.status` is typed as `"pending" | "in_progress" | "done" | "all" | undefined`. The contracts package exports `TODO_STATUSES` constants and a `TodoStatus` type that covers the same values. The local literal union must be manually kept in sync — if a new status is added to the backend, the local type will silently allow stale values.
- **Suggestion**: Import `TodoStatus` from `@slopweaver/contracts` and use `TodoStatus | "all" | undefined` (if `"all"` is a UI-only sentinel not in the contract type). This ensures that new contract statuses are automatically reflected in the query params type.
- **Evidence**: `status?: "pending" | "in_progress" | "done" | "all";` in `TodoQueryParams` instead of `TodoStatus | "all"`.

### Finding 7: Local `ToolCallStatus` type in `tool-label.utils.ts` duplicates `@/types/assistant`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/tool-label.utils.ts:17-24`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `tool-label.utils.ts` defines a local `ToolCallStatus = "pending" | "running" | "completed" | "error"` type. The same (or structurally identical) `ToolCallStatus` type is already defined in `@/types/assistant`. Having two definitions means callers from different import paths receive nominally different types, which can cause type errors when values from one source are passed to a function expecting the other. Additionally, `TOOL_LABELS: Record<string, string>` accepts any string key — if the tool names are a known set, using the known union as the key type would catch missing or misspelled labels at compile time.
- **Suggestion**: Remove the local `ToolCallStatus` definition and import it from `@/types/assistant`. For `TOOL_LABELS`, if the tool names are a defined constant set (e.g. `keyof typeof TOOL_CALL_NAMES`), use `Record<ToolCallName, string>` or `Partial<Record<ToolCallName, string>>` as the map type.
- **Evidence**: `type ToolCallStatus = "pending" | "running" | "completed" | "error";` defined locally in `tool-label.utils.ts` alongside an identical definition in `@/types/assistant`.
