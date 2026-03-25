# Audit: apps-api-src-application-chat

**Files audited**

- `apps/api/src/application/chat/errors/chat.errors.ts`
- `apps/api/src/application/chat/utils/error-mapper.ts`

## Findings

No violations found.

The three user-facing message strings in `chat.errors.ts` are plain and direct:

- `Conversation "${conversationId}" not found`
- `Insufficient actions for ${action}. Required: ${requiredActions} actions.`
- `Chat message with ID "${chatId}" not found`

`error-mapper.ts` contains no user-facing message strings; it maps error codes to HTTP status codes only.
