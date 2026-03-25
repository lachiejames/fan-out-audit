# Audit: apps-api-src-application-auth

**Files inspected**: 2
**Findings**: 0

## Summary

Both files are internal backend code. User-visible strings are limited to three short, functional messages:

- `"Unknown auth error occurred"` (exhaustive-switch fallback in error-mapper.ts)
- `"${resource} with ID \"${id}\" not found"` (notFound factory)
- `"${resource} not found"` (notFound factory, id-less variant)

All three are terse and technical. No AI writing tropes are present.

## No findings
