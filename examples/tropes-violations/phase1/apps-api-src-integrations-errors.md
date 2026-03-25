# Tropes Audit: apps/api/src/integrations/errors

## Files Audited

- `apps/api/src/integrations/errors/error-mapper.ts`
- `apps/api/src/integrations/errors/integration.errors.ts`

## Findings

No findings.

All string content in these files is internal: discriminant code strings (`"INTEGRATION_NOT_FOUND"`, `"BUDGET_EXCEEDED"`, etc.), developer-facing error messages surfaced in logs and API error responses consumed by the frontend error handler, and a fallback mapper string (`"Unknown error: ..."`). None of this text is rendered directly in the UI as user-facing copy. No AI writing tropes apply.
