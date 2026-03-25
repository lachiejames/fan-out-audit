# Audit: apps-api-src-application-knowledge-sources

## Files Audited

- `apps/api/src/application/knowledge-sources/errors/knowledge-sources.errors.ts`
- `apps/api/src/application/knowledge-sources/utils/error-mapper.ts`

## Findings

No findings.

Both files are internal backend infrastructure with no user-facing text. `knowledge-sources.errors.ts` defines discriminated union error types and factory functions that produce developer-facing error strings (used in API responses and logs). `error-mapper.ts` maps those error types to HTTP status codes. Neither file contains marketing copy, UI strings, or any prose that could carry AI writing tropes.
