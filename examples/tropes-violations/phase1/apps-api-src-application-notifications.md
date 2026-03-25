# Audit: apps-api-src-application-notifications

**Files inspected**: 6
**Findings**: 1

## Summary

Five of the six files contain no user-facing text at all (pure TypeScript infrastructure: module wiring, type definitions, utility functions, error factory functions, error mapper). One file, `notification-scheduler.service.ts`, generates three notification messages that are delivered directly to users. One minor issue was found in those messages.

## Findings

### 1. Filler / padding phrase in notification body

**File**: `apps/api/src/application/notifications/services/notification-scheduler.service.ts`
**Line**: 75
**Category**: Filler transitions / padding

**Flagged text**:

```
`Your snoozed task "${truncate({ maxLength: 50, str: todo.content })}" is ready for attention.`
```

"Ready for attention" is soft filler. The task woke up from snooze -- it is simply pending again. "is ready for attention" adds no information the user does not already know from the notification title "Snoozed Task Reminder". A direct rewrite would be: `Your snoozed task "..." is now active.` or simply drop the trailing clause: `"..." has woken up.`

**Other notification messages for reference** (no issues found):

- `"${truncate(...)}" is due within 24 hours.` -- plain, direct.
- `"${truncate(...)}" is now overdue.` -- plain, direct.
