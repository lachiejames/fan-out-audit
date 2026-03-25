# Audit: apps-api-src-application-account-deletion

**Files inspected**: 2
**Findings**: 0

## Summary

Both files are internal backend TypeScript. The only user-facing strings are five terse error messages in `account-deletion.errors.ts` ("Email address is invalid", "Account deletion request not found", "Deletion token has expired", "Deletion token is invalid", "Deletion token has already been used"). None exhibit AI writing tropes. The error-mapper file contains no user-facing text at all.

## No findings

All user-facing strings are short, factual error messages free of AI writing tropes; the error-mapper file contains only numeric HTTP status codes and internal identifiers with no prose for users to read.
