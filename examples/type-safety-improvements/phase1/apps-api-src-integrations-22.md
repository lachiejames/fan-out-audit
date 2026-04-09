# Type Safety Audit — apps/api/src/integrations (Batch 22)

**Files inspected**: 8  
**Findings**: 6

## Summary

The Jira utility files are largely well-structured. Batch 22 spans the remaining Jira utils and the start of the Linear integration. Key issues: the `jira-sync.utils.ts` `JiraApiIssueFields` type uses an index signature that allows arbitrary field access; the Linear plugin casts the `parseCursorState` result; the `linear-sync-fetch.service.ts` `ResolvedIssue` interface uses inline dynamic import syntax for SDK types; and Linear services use bracket-access on `pageInfo` where typed dot-notation is available. The `linear-oauth.service.ts` makes a raw JSON cast from an in-house GraphQL endpoint.

## Findings

---

### Finding 1

**File**: `apps/api/src/integrations/platforms/jira/utils/jira-sync.utils.ts:44`  
**Category**: `Record<string, ...>` weakening  
**Impact**: Low

**Description**: `JiraApiIssueFields` defines `[key: string]: unknown` alongside named fields like `summary`, `status`, `assignee`. This index signature defeats exhaustive property checking on known fields and allows arbitrary key access to silently return `unknown`.

**Suggestion**: Remove the index signature from `JiraApiIssueFields`. Pass custom fields as a separate `customFields: Record<string, unknown>` property rather than polluting the main type with an open index signature.

**Evidence**:

```typescript
// jira-sync.utils.ts:44
interface JiraApiIssueFields {
  summary: string;
  // ... named fields ...
  [key: string]: unknown; // defeats exhaustive checking
}
```

---

### Finding 2

**File**: `apps/api/src/integrations/platforms/linear/linear.plugin.ts:263`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: The result of `parseCursorState({ value: params.resumeState })` is cast to `LinearCursorState`. `parseCursorState` returns `unknown`, so the cast is effectively a "trust me" assertion with no runtime validation.

**Suggestion**: Add a Zod schema or type guard for `LinearCursorState` and validate the output of `parseCursorState` before assigning, consistent with how other cursor states are handled.

**Evidence**:

```typescript
// linear.plugin.ts:263
syncParams.resumeState = parseCursorState({ value: params.resumeState }) as LinearCursorState;
```

---

### Finding 3

**File**: `apps/api/src/integrations/platforms/linear/services/linear-sync-fetch.service.ts:61`  
**Category**: SDK type duplication / Missing strict typing  
**Impact**: Low

**Description**: The `ResolvedIssue` interface uses inline dynamic import syntax (`import("@linear/sdk").WorkflowState`, `import("@linear/sdk").Cycle`, `import("@linear/sdk").Project`) instead of top-level named imports. This is non-idiomatic, harder to read, and makes it harder to search for usages of these SDK types.

**Suggestion**: Add named imports at the top of the file: `import type { WorkflowState, Cycle, Project } from "@linear/sdk";` and use them directly in the interface.

**Evidence**:

```typescript
// linear-sync-fetch.service.ts:59-68
export interface ResolvedIssue {
  issue: Issue;
  state: import("@linear/sdk").WorkflowState | undefined;
  cycle: import("@linear/sdk").Cycle | undefined;
  project: import("@linear/sdk").Project | undefined;
  // ...
}
```

---

### Finding 4

**File**: `apps/api/src/integrations/platforms/linear/services/linear-sync-fetch.service.ts:97`  
**Category**: Missing strict typing  
**Impact**: Medium

**Description**: The Zod cache validation schemas for `User` and `Issue` use `z.custom<User>(isRecord)` and `z.custom<Issue>(isRecord)`. `isRecord` only checks `typeof value === "object" && value !== null`. This means any plain object passes as a valid `User` or `Issue` — fields like `id`, `title`, `state` are never validated, so a corrupted or outdated cache entry will be accepted as valid without error.

**Suggestion**: Replace `z.custom<User>(isRecord)` with a proper Zod schema that validates the minimum required fields (e.g., `id: z.string()`). For SDK objects that are too complex to fully validate, at minimum check the `id` and `__typename` fields.

**Evidence**:

```typescript
// linear-sync-fetch.service.ts:96-99
const isRecord = (value: unknown): value is Record<string, unknown> => typeof value === "object" && value !== null;
const linearViewerSchema: z.ZodType<User> = z.custom<User>(isRecord); // any object passes
const linearIssueSchema: z.ZodType<Issue> = z.custom<Issue>(isRecord); // any object passes
const linearIssuesSchema: z.ZodType<Issue[]> = z.array(linearIssueSchema);
```

---

### Finding 5

**File**: `apps/api/src/integrations/platforms/linear/services/linear-sync-fetch.service.ts:229`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `response.pageInfo` (where `response` is a `LinearFetch<IssueConnection>`) is cast to `{ endCursor?: string; hasNextPage?: boolean }`. The `@linear/sdk` `PageInfo` type exports `hasNextPage: boolean` and `endCursor?: string | undefined` as proper named properties — dot-notation access is available and safer.

**Suggestion**: Remove the cast and access `response.pageInfo.endCursor` and `response.pageInfo.hasNextPage` directly.

**Evidence**:

```typescript
// linear-sync-fetch.service.ts:229
const pageInfo = response.pageInfo as { endCursor?: string; hasNextPage?: boolean };
```

---

### Finding 6

**File**: `apps/api/src/integrations/platforms/linear/services/linear-oauth.service.ts:547`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: The response from an internal GraphQL endpoint is cast inline to a deeply nested shape `{ data: { organization: { users: { nodes: {...}[] } } } }` without any runtime validation. If the endpoint returns an error object or unexpected shape, the cast succeeds and subsequent destructuring will throw at runtime.

**Suggestion**: Define a Zod schema for this response shape and use `schema.parse()` or `schema.safeParse()` before accessing nested properties.

**Evidence**:

```typescript
// linear-oauth.service.ts:547 (approximate)
const json = (await response.json()) as {
  data: { organization: { users: { nodes: { id: string; name: string; email: string }[] } } };
};
```
