# Slice 195: packages/ui/src/organisms/chat (8 files)

Files audited:

- `packages/ui/src/organisms/chat/code-block.tsx`
- `packages/ui/src/organisms/chat/code-block-context.tsx`
- `packages/ui/src/organisms/chat/code-block-copy-button.tsx`
- `packages/ui/src/organisms/chat/controls.tsx`
- `packages/ui/src/organisms/chat/response.tsx`
- `packages/ui/src/organisms/chat/shimmer.tsx`
- `packages/ui/src/organisms/chat/tool-content.tsx`
- `packages/ui/src/organisms/chat/tool-header.tsx`

---

## code-block.tsx, code-block-context.tsx, code-block-copy-button.tsx

No user-facing strings. The copy button shows no text label (icon only). No violations.

---

## controls.tsx

React Flow canvas controls component wrapper. No user-facing strings. No violations.

---

## response.tsx

Memoized markdown renderer. No authored strings. No violations.

---

## shimmer.tsx

Shimmer text component. Renders whatever `children` is passed — no authored copy. No violations.

---

## tool-content.tsx

Collapsible wrapper. No user-facing strings. No violations.

---

## tool-header.tsx

**User-facing strings in status badge labels:**

```typescript
const labels: Record<ToolUIPart["state"], string> = {
  "approval-requested": "Awaiting Approval",
  "approval-responded": "Approved",
  "input-available": "Running",
  "input-streaming": "Pending",
  "output-available": "Completed",
  "output-denied": "Denied",
  "output-error": "Error",
};
```

These are functional status labels for tool call UI. No AI writing tropes — they describe machine states accurately and specifically.

---

## Findings

No AI writing tropes found in this slice.
