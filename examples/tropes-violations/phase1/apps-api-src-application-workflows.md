# Audit: apps-api-src-application-workflows

**Files audited:**

- `apps/api/src/application/workflows/errors/workflow.errors.ts`
- `apps/api/src/application/workflows/utils/error-mapper.ts`

## No findings

Both files are internal backend infrastructure with no user-facing prose. The error strings in `workflow.errors.ts` are terse technical messages surfaced only in API error responses, not end-user UI. The error-mapper file contains no prose at all, only status code routing logic. No AI writing tropes present.
