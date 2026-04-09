# Audit: apps-app-src-components-19

**Files inspected**:

- `apps/app/src/components/queue/queue-preview-router.ts`
- `apps/app/src/components/queue/queue-preview.types.ts`
- `apps/app/src/components/queue/queue-send-state.ts`
- `apps/app/src/components/queue/queue-slide-over-calendar-edit-fields.tsx`
- `apps/app/src/components/queue/queue-slide-over-edit-section.tsx`
- `apps/app/src/components/queue/queue-slide-over-footer.tsx`
- `apps/app/src/components/queue/queue-slide-over-header.tsx`
- `apps/app/src/components/queue/queue-slide-over-terminal-info.tsx`

**Findings**: 3

---

## Summary

Three findings. `queue-preview-router.ts` uses `workItem.type as "add_reaction" | "delete_event"` for the `MinimalActionPreview` discriminant — the cast could be eliminated with a proper type guard. `queue-slide-over-edit-section.tsx` inlines `"draft" | "sending" | "sent" | "error"` instead of importing the already-defined `QueueSendState` type. `queue-slide-over-terminal-info.tsx` casts `workItem.status as keyof typeof TERMINAL_CONFIG` where using `WorkItemStatus` narrowing would be safer.

---

## Findings

### Finding 1: `workItem.type as "add_reaction" | "delete_event"` cast in `queue-preview-router.ts`

- **File**: `apps/app/src/components/queue/queue-preview-router.ts`
- **Category**: type-cast
- **Impact**: low
- **Description**: At line 73, `actionType: workItem.type as "add_reaction" | "delete_event"` forces a cast to satisfy the `MinimalActionPreview.actionType` type. This is needed because `workItem.type` is typed as `string` on `QueueItem`. If `workItem.type` were narrowed from `string` to `WorkItemType`, the discriminant would either match exactly or require a proper guard, eliminating the cast.
- **Suggestion**: Narrow `QueueItem.type` from `string` to `WorkItemType` (from `@slopweaver/contracts`). After that change, the `case "minimal"` branch where `MINIMAL_ACTION_TYPES.has(actionType)` already ensures the type is one of the minimal types, so the cast becomes unnecessary.
- **Evidence**:
  ```typescript
  // line 73
  actionType: workItem.type as "add_reaction" | "delete_event",
  ```

---

### Finding 2: Inline `"draft" | "sending" | "sent" | "error"` literal instead of `QueueSendState` in `queue-slide-over-edit-section.tsx`

- **File**: `apps/app/src/components/queue/queue-slide-over-edit-section.tsx`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: At line 51, `setSendState: Dispatch<SetStateAction<"draft" | "sending" | "sent" | "error">>` inlines the send state union instead of using the already-exported `QueueSendState` type from `queue-send-state.ts`. If the union changes, this prop definition is not automatically updated.
- **Suggestion**: Import `QueueSendState` from `@/components/queue/queue-send-state` and use `Dispatch<SetStateAction<QueueSendState>>`.
- **Evidence**:
  ```typescript
  // line 51
  setSendState: Dispatch<SetStateAction<"draft" | "sending" | "sent" | "error">>;
  // Should be:
  setSendState: Dispatch<SetStateAction<QueueSendState>>;
  ```

---

### Finding 3: `workItem.status as keyof typeof TERMINAL_CONFIG` in `queue-slide-over-terminal-info.tsx`

- **File**: `apps/app/src/components/queue/queue-slide-over-terminal-info.tsx`
- **Category**: type-cast
- **Impact**: low
- **Description**: At line 54, `TERMINAL_CONFIG[workItem.status as keyof typeof TERMINAL_CONFIG]` casts `workItem.status` (which is `WorkItemStatus` or `string`) to bypass TypeScript's index check. If `workItem.status` were narrowed to `WorkItemStatus` and `TERMINAL_CONFIG` were typed `Partial<Record<WorkItemStatus, ...>>`, the cast would be unnecessary.
- **Suggestion**: Type `TERMINAL_CONFIG` as `Partial<Record<WorkItemStatus, ...>>` and access it as `TERMINAL_CONFIG[workItem.status]`. The null check `if (!config) return null` already handles non-terminal statuses.
- **Evidence**:
  ```typescript
  // line 54
  const config = TERMINAL_CONFIG[workItem.status as keyof typeof TERMINAL_CONFIG];
  ```
