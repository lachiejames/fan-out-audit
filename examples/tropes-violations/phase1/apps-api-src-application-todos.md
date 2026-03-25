# Audit: apps-api-src-application-todos

Files reviewed:

- `apps/api/src/application/todos/errors/todo.errors.ts`
- `apps/api/src/application/todos/errors/todo-extraction.errors.ts` (does not exist)
- `apps/api/src/application/todos/utils/error-mapper.ts`

## No findings

All content in these files is internal: TypeScript error type definitions, discriminated union error factories, and an HTTP status code mapping table. No user-facing strings are present beyond terse technical error messages used in server logs and API responses (`"Failed to insert todo"`, `"At least one field must be provided for update"`). None of these strings are rendered in the UI or surfaced as marketing/product copy. No AI writing tropes detected.

Note: `todo-extraction.errors.ts` did not exist at the time of this audit.
