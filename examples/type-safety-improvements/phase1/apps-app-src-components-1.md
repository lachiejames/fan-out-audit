# Audit: apps-app-src-components-1

**Files inspected**: 8
**Findings**: 3

## Summary

The component files in this slice are mostly well-typed with explicit prop interfaces and no `any` usage. The notable findings are: a locally-defined `ToolState` type that likely duplicates the AI SDK v6's `ToolUIPart` state shape, props declared in an interface but never destructured in the function body (`approvalId` and `toolName` in `ai-chat-tool-approval-actions.tsx`), and a `FileValidationResult` discriminated union that may overlap with an existing `ValidationResult` type in `@slopweaver/ui`.

## Findings

### Finding 1: `ToolState` type in `ai-chat-message.utils.ts` may duplicate AI SDK v6's `ToolUIPart` state

- **File**: `apps/app/src/components/ai-chat-message.utils.ts:4-10`
- **Category**: sdk-type-duplication
- **Impact**: medium
- **Description**: `ToolState` is defined locally as `"partial-call" | "call" | "result"`. The AI SDK v6 (`ai` / `@ai-sdk/react`) exports a `ToolUIPart` type that includes a `state` field with these exact values. If the SDK type is available, defining a local union creates a risk of divergence if the SDK adds or renames states in a future release.
- **Suggestion**: Import the state type from the AI SDK (e.g. `ToolUIPart["state"]` or a named export if one exists) instead of redeclaring it locally. If the SDK type is unavailable at the `@slopweaver/app` layer, document why the local definition is necessary.
- **Evidence**:
  ```ts
  export type ToolState = "partial-call" | "call" | "result";
  ```

### Finding 2: `approvalId` and `toolName` declared in props but never destructured in `ai-chat-tool-approval-actions.tsx`

- **File**: `apps/app/src/components/ai-chat-tool-approval-actions.tsx:6-13`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `ToolApprovalActionsProps` declares `approvalId: string` and `toolName: string`, but neither field is destructured or used in the function body. These are dead props — passing them has no effect. This is either a bug (the fields were meant to be used) or the interface should be trimmed.
- **Suggestion**: If `approvalId` and `toolName` are needed (e.g. for future analytics or display), wire them up in the component body. If they are unused, remove them from the interface to prevent callers from being misled into thinking they affect behaviour.
- **Evidence**:
  ```ts
  interface ToolApprovalActionsProps {
    approvalId: string;
    toolName: string;
    onApprove: () => void;
    onDeny: () => void;
  }
  // approvalId and toolName are not destructured or referenced in the function body
  ```

### Finding 3: `FileValidationResult` in `ai-chat-widget-file-validation.utils.ts` may overlap with `ValidationResult` from `@slopweaver/ui`

- **File**: `apps/app/src/components/ai-chat-widget-file-validation.utils.ts:80`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: `FileValidationResult` is a locally-defined discriminated union: `{ valid: true } | { valid: false; errorTitle: string; errorDescription: string }`. The `@slopweaver/ui` package exports a `ValidationResult` type used in `media-validation.utils.ts`. If the shapes are compatible, using the shared type would reduce duplication and allow consumers to work with a single validation result contract.
- **Suggestion**: Check whether `@slopweaver/ui`'s `ValidationResult` is structurally compatible with `FileValidationResult`. If so, replace the local definition with an import from `@slopweaver/ui`. If the shapes differ, document why a separate type is needed.
- **Evidence**:
  ```ts
  export type FileValidationResult = { valid: true } | { valid: false; errorTitle: string; errorDescription: string };
  ```

## No additional findings

`action-meter-mini.tsx`, `ai-chat-message.tsx`, `ai-chat-timeline.tsx`, `ai-chat-widget-file-preview.tsx`, and `ai-chat-widget-panel.tsx` are clean: they use explicit prop interfaces, no `any`, and no unsafe casts.
