# Audit: apps-api-src-application-insights

**Files audited**:

- `apps/api/src/application/insights/errors/insight.errors.ts`
- `apps/api/src/application/insights/utils/error-mapper.ts`

## No findings

Both files contain only internal backend infrastructure: a discriminated union error type definition and an HTTP error mapper factory call. There is no user-facing text, no error messages surfaced to end users, and no product or marketing copy. The single user-visible string (`"Unknown error"` in the fallback of `error-mapper.ts`) is a generic developer-facing fallback that never reaches the UI in normal operation.

Nothing to fix.
