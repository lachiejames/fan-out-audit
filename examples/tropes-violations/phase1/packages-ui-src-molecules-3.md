# Tropes Audit: packages/ui/src/molecules — Slice 3

Files reviewed:

- `packages/ui/src/molecules/collapsible.tsx`
- `packages/ui/src/molecules/command.tsx`
- `packages/ui/src/molecules/dialog.tsx`
- `packages/ui/src/molecules/dropdown-menu.tsx`
- `packages/ui/src/molecules/empty-state.tsx`
- `packages/ui/src/molecules/enhanced-alert.tsx`

---

## collapsible.tsx

No user-visible text. No findings.

---

## command.tsx

**CommandDialog default props (line 35-36):**

> title default: `"Command Palette"`
> description default: `"Search for a command to run..."`

Plain functional defaults. No trope language.

---

## dialog.tsx

**Close button sr-only text (line 292):**

> `"Close"`

Accessibility text. No findings.

---

## dropdown-menu.tsx

No user-visible text. No findings.

---

## empty-state.tsx

**JSDoc example (line 48-52):**

> `title="No messages"` / `description="Your inbox is empty. Connect an integration to get started."` / `action={{ label: "Connect Slack" }}`

Plain functional empty-state copy. No AI writing tropes.

**Use-cases comment (lines 31-38):**

> `"Empty inbox: No messages"` / `"Empty TODO list: No tasks"` / `"Empty search results: No matches found"` / `"Empty conversation list: No conversations"` / `"Error states: Failed to load"`

Internal developer documentation, not user-facing copy. No findings.

---

## enhanced-alert.tsx

**JSDoc example (line 29-35):**

> `title="Code Agent Failed"` / labels `"Retry"`, `"View Logs"`

No trope language. Functional labels. No findings.

**Variant comments (lines 39-43):**

> `"danger: API errors, agent failures (red)"` / `"warning: Low confidence, potential issues (yellow)"` / `"info: Tips, limited data (blue)"` / `"success: Completed actions (green)"`

Internal developer documentation, not user-facing copy. No findings.

---

## Summary

No AI writing tropes found in this slice.
