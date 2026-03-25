# Tropes Audit: packages/ui/src/molecules — Slice 1

Files reviewed:

- `packages/ui/src/molecules/accordion.tsx`
- `packages/ui/src/molecules/alert.tsx`
- `packages/ui/src/molecules/alert-dialog.tsx`
- `packages/ui/src/molecules/alert-dialog.stories.tsx`
- `packages/ui/src/molecules/blankslate.tsx`
- `packages/ui/src/molecules/breadcrumbs.tsx`

---

## accordion.tsx

No user-visible text. Pure structural/layout component. No findings.

---

## alert.tsx

No user-visible text. Pure structural component. No findings.

---

## alert-dialog.tsx

**JSDoc example text (line 21):**

> `"Are you sure?"`

Generic but not AI-trope language. Standard destructive-action confirmation pattern. No findings.

---

## alert-dialog.stories.tsx

**Default story (line 41):**

> `"Are you absolutely sure?"`

Intensifier "absolutely" is mildly dramatic but this is standard boilerplate for delete confirmation dialogs. Not an AI writing trope.

**Default story description (line 43-44):**

> `"This action cannot be undone. This will permanently delete your account and remove your data from our servers."`

Standard destructive-action copy. No trope language.

**Destructive story (line 65-67):**

> `"Are you sure you want to delete your account? All of your data will be permanently removed. This action cannot be undone."`

Same pattern. No findings.

**Simple story (line 88-89):**

> `"Your session is about to expire. Would you like to stay signed in?"`

Plain functional copy. No findings.

---

## blankslate.tsx

**JSDoc example text (line 22):**

> `"No messages yet"` / `"When you receive messages, they'll appear here"`

Plain functional empty-state copy. No AI writing tropes.

---

## breadcrumbs.tsx

No user-visible text. No findings.

---

## Summary

No AI writing tropes found in this slice.
