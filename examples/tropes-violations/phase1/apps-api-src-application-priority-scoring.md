# Audit: apps-api-src-application-priority-scoring

**Files reviewed:**

- `apps/api/src/application/priority-scoring/errors/priority-scoring.errors.ts`
- `apps/api/src/application/priority-scoring/utils/error-mapper.ts`

## No findings

Both files are internal backend infrastructure with no user-facing text. All strings are developer/API-level error messages (`Content not found: ${contentId}`, `VIP contact already exists: ${contactIdentifier}`, `VIP contact not found: ${contactId}`, `Unknown error: ${JSON.stringify(error)}`). These surface in HTTP response bodies consumed by the frontend, not rendered directly in UI copy.

No AI writing tropes present.
