# Audit: apps-app-src-types

**Files inspected**: 1
**Findings**: 3

## Summary

`assistant.ts` is a well-structured types file. The main issues are `Record<string, unknown>` used for `ToolCall.args` (unavoidable to some extent, but improvable), `result?: unknown` on `ToolCall` which forces callers to cast before use, and an `args` field compared by reference (`ai.args !== bi.args`) in the equality util that would never be equal for two independently constructed objects.

---

## Findings

### Finding 1: `ToolCall.args` typed as `Record<string, unknown>`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/types/assistant.ts:31`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `args: Record<string, unknown>` for tool call arguments is a deliberate design choice when tools are dynamic — each tool has a different argument schema. However, since `ToolCall.name` identifies the tool, it would be possible to create a discriminated union `ToolCall<Name extends ToolName>` with a typed `args` per tool name. The generic approach would only apply to known/static tools.
- **Suggestion**: For now this is acceptable given the dynamic nature of AI tool calls. Consider creating a narrowed `KnownToolCall` discriminated union type for the known/static tools listed in `TOOL_LABELS` (e.g. `SearchWorkspaceToolCall`, `CreateTodoToolCall`) that consumers can use where the tool name is known statically.
- **Evidence**: `args: Record<string, unknown>;`

### Finding 2: `ToolCall.result` typed as `unknown`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/types/assistant.ts:32`
- **Category**: any-usage
- **Impact**: low
- **Description**: `result?: unknown | undefined` forces every consumer to perform type-narrowing before using the result. The AI SDK tool result type is `unknown` by default for dynamic tools, but for known tools the result type is part of the tool definition. Where results are displayed (e.g. in `AIToolRow`), they are converted to string via `JSON.stringify` or `String()`.
- **Suggestion**: For display purposes, define a helper `formatToolResult(result: unknown): string` (already exists as `formatToolCallResult` in `message-comparison.utils.ts`). For computation, leave as `unknown` but document that consumers must narrow before use. This is already the standard pattern and no change is strictly required.
- **Evidence**: `result?: unknown | undefined;`

### Finding 3: Re-exporting `Citation` and `CitationPlatform` from contracts in assistant.ts

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/types/assistant.ts:14`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: `assistant.ts` imports and immediately re-exports `Citation` and `CitationPlatform` from `@slopweaver/contracts`. Per `code-organization.md`, convenience re-exports are banned ("just import from source"). Consumers of `Citation` from `@/types/assistant` should import directly from `@slopweaver/contracts`.
- **Suggestion**: Remove the re-exports from `assistant.ts`. Update any files that `import { Citation } from "@/types/assistant"` to `import type { Citation } from "@slopweaver/contracts"` directly.
- **Evidence**: `export type { Citation, CitationPlatform };` on line 14, imported via `import type { Citation, CitationPlatform, UiPart } from "@slopweaver/contracts";`
