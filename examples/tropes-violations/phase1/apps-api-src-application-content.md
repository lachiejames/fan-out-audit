# Audit: apps-api-src-application-content

**Files reviewed:**

- `apps/api/src/application/content/errors/content.errors.ts`
- `apps/api/src/application/content/utils/error-mapper.ts`

## Findings

No violations found.

Both files contain only internal API error messages (JSON error responses for HTTP consumers). The strings are terse and technical: `Content with ID "${contentId}" not found`, `Invalid source "${source}". Valid sources: ...`. No AI writing tropes present.
