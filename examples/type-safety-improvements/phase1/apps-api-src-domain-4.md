# Audit: apps-api-src-domain-4

**Files inspected**: 8
**Findings**: 4

## Summary

The platform port files in this batch are generally well-typed. Key findings: `IPlatformSearchPort` defines GitHub issue/PR reaction shapes inline with duplicate structures; `PlatformSearchError.details` is `Record<string, unknown>`; `IWorkItemExecutorPort` has `canUndo(actionType: string)` with a positional param instead of a named param; and `ILoggerPort` uses `unknown` for message parameters which is acceptable but worth noting.

## Findings

### Finding 1: Duplicate GitHub reaction shape defined twice inline

- **File**: `apps/api/src/domain/ports/integrations/platform-search.port.ts:87-97` and `110-120`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: The reactions object shape `{ total: number; thumbsUp: number; thumbsDown: number; laugh: number; hooray: number; confused: number; heart: number; rocket: number; eyes: number }` is defined identically inline in both `GitHubIssueResult.issue.reactions` (line ~88) and `GitHubIssueResult.comments[].reactions` (line ~112). If the GitHub reactions API adds a new emoji, both inline types need updating independently.
- **Suggestion**: Extract to a shared interface `GitHubReactions` and reference it in both locations.
- **Evidence**:

```typescript
// In GitHubIssueResult.issue:
reactions: {
  total: number;
  thumbsUp: number;
  thumbsDown: number;
  laugh: number;
  hooray: number;
  confused: number;
  heart: number;
  rocket: number;
  eyes: number;
}

// In GitHubIssueResult.comments[]:
reactions: {
  total: number;
  thumbsUp: number;
  thumbsDown: number;
  laugh: number;
  hooray: number;
  confused: number;
  heart: number;
  rocket: number;
  eyes: number;
}
```

### Finding 2: `PlatformSearchError.details` typed as `Record<string, unknown>`

- **File**: `apps/api/src/domain/ports/integrations/platform-search.port.ts:27`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `PlatformSearchError` constructor accepts `details?: Record<string, unknown>`. This is a catch-all for platform-specific error context. While flexibility is needed here, it means error handlers can never rely on specific keys being present. At minimum the most common details keys (e.g., `statusCode`, `responseBody`) should be documented via JSDoc.
- **Suggestion**: Add JSDoc `@param details` listing the most common keys. Optionally define a `PlatformSearchErrorDetails` interface with known optional fields + an index signature.
- **Evidence**:

```typescript
export class PlatformSearchError extends Error {
  constructor(
    public readonly type: PlatformSearchErrorType,
    message: string,
    public readonly platform?: string,
    public readonly details?: Record<string, unknown>,
  ) { ... }
}
```

### Finding 3: `IWorkItemExecutorPort.canUndo` uses positional param instead of named param

- **File**: `apps/api/src/domain/ports/integrations/work-item-executor.port.ts:86`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `abstract canUndo(actionType: string): boolean` uses a positional parameter. The codebase-wide rule requires named object params for all functions with 1+ parameters. This is an abstract method signature on an abstract class.
- **Suggestion**: Change to `abstract canUndo({ actionType }: { actionType: string }): boolean` to match the named-params convention.
- **Evidence**:

```typescript
abstract canUndo(actionType: string): boolean;
```

### Finding 4: `NotionQueryDatabaseCommand.filter` typed as `Record<string, unknown>`

- **File**: `apps/api/src/domain/ports/integrations/project-management.port.ts:122`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `NotionQueryDatabaseCommand.filter?: Record<string, unknown>` is used to pass a Notion database filter. The Notion SDK exports a `QueryDatabaseParameters['filter']` union type that describes the exact structure. Since this is a domain port intentionally decoupled from the Notion SDK, the loose type is architecturally justified, but a JSDoc comment linking to the Notion filter structure would help callers.
- **Suggestion**: Add a JSDoc comment `@see https://developers.notion.com/reference/post-database-query-filter` describing the expected filter shape. No code change required.
- **Evidence**:

```typescript
export interface NotionQueryDatabaseCommand extends BaseCommand {
  databaseId: string;
  filter?: Record<string, unknown>;
  sorts?: { property: string; direction: "ascending" | "descending" }[];
  limit?: number;
}
```
