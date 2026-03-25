# Audit: apps-api-src-application-suggested-actions

## Files audited

- `apps/api/src/application/suggested-actions/errors/suggested-actions.errors.ts`
- `apps/api/src/application/suggested-actions/utils/error-mapper.ts`

## Findings

No findings.

Both files are internal TypeScript with no user-facing text. `suggested-actions.errors.ts` defines domain error types and factory functions; `error-mapper.ts` maps error codes to HTTP status integers. The only string content is a developer-facing error message (`Content with ID "${contentId}" not found`) used in server logs and API error responses, not in any UI copy.
