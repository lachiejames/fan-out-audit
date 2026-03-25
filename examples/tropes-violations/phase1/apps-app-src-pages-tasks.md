# Trope Audit: apps/app/src/pages/tasks

Files audited:

- `loading.tsx`
- `page.tsx`
- `tasks-board-view.tsx`
- `tasks-calendar-panel.tsx`
- `tasks-list-view.tsx` (file does not exist at this path)

## Findings

### page.tsx

**"Smart Sort"** (line 338, sort option label)
Category: AI-flavored label for a mundane feature.
"Smart" is a common AI writing trope used when the actual sorting logic is unexplained to the user. The option lives alongside concrete alternatives ("Newest First", "Priority") so the vagueness stands out. Should be renamed to describe what it actually does (e.g., "Recommended" if it is a relevance sort, or "By priority + date" if that is the algorithm).

**"Add a task..."** placeholder (line 374)
Category: Ellipsis trailing off.
Minor. The ellipsis in placeholder text is a common AI-generated filler tic. Prefer a direct label: "Add a task" or "Task title".

No other user-visible copy tropes found. The rest of the file is structural logic, filter labels ("All Categories", "Urgent", "Important"), and status labels ("To Do", "In Progress", "Done") -- all concrete and accurate.

### tasks-board-view.tsx

No findings. Column headers ("To Do", "In Progress", "Done") are direct. No marketing copy.

### tasks-calendar-panel.tsx

**"Coming Up"** section header (line 177, `<h4>`)
Category: Vague section title.
"Coming Up" is imprecise. "Upcoming Due Tasks" (or just "Due Soon") states what is actually shown. The surrounding code confirms it is a list of tasks with due dates sorted by urgency.

**"No upcoming due tasks."** (line 194)
Clean and specific. No issue.

### loading.tsx

No findings. Pure skeleton/shimmer markup with no user-visible copy.

### tasks-list-view.tsx

File not found at the audited path. Nothing to audit.

## Summary

| File                     | Violations    |
| ------------------------ | ------------- |
| loading.tsx              | 0             |
| page.tsx                 | 2 (minor)     |
| tasks-board-view.tsx     | 0             |
| tasks-calendar-panel.tsx | 1 (minor)     |
| tasks-list-view.tsx      | N/A (missing) |

Total: 3 minor findings. No severe marketing-speak or AI hype tropes.
