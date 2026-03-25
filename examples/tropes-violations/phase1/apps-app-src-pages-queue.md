# Trope Audit: apps/app/src/pages/queue

Files audited:

- `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/queue/loading.tsx`
- `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/queue/page.tsx`

---

## loading.tsx

No user-facing text. Renders a skeleton layout with animated placeholder elements only.

**Findings:** None.

---

## page.tsx

### User-facing strings inventory

| Location                              | Text                                 |
| ------------------------------------- | ------------------------------------ |
| View toggle button                    | "Queue"                              |
| View toggle button                    | "Audit log"                          |
| Toast (approve all success, singular) | "1 action executed"                  |
| Toast (approve all success, plural)   | "{n} actions executed"               |
| Toast (approve all failure, all)      | "All actions failed to execute"      |
| Toast (approve all failure, partial)  | "{n} action(s) failed; {n} executed" |
| Toast (approve all catch)             | "Some actions failed to execute"     |
| Toast (undo success)                  | "Action undone"                      |
| Toast (undo failure)                  | "Failed to undo action"              |
| Toast (audit item load failure)       | "Failed to load work item"           |

### Analysis

All strings are operational status messages. No marketing copy, no AI tropes, no em dashes, no hype words. Toast messages are plain and informational.

The page itself contains no headings, descriptions, or promotional text. The substantive UI text (empty states, filter labels, card content) lives in sub-components (`QueueEmptyState`, `QueueFilterBar`, `QueueCard`) which are outside this slice's scope.

**Findings:** None.
