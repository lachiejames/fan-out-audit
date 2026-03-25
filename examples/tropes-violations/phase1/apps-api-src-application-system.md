# Audit: apps-api-src-application-system

Files reviewed:

- `apps/api/src/application/system/context-status/errors/context-status.errors.ts`
- `apps/api/src/application/system/context-status/utils/error-mapper.ts`

No findings.

Both files contain internal developer-facing code only. The errors file defines a discriminated union error type and a factory function where the `message` string is supplied by callers at the point of use, not hardcoded here. The error-mapper file maps the `INTERNAL_ERROR` code to HTTP status 500. Neither file contains user-facing copy, fixed error message strings, or any prose that could carry AI writing tropes.
