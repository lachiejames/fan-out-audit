# Audit: apps-api-src-application-training

Files audited:

- `apps/api/src/application/training/errors/training.errors.ts`
- `apps/api/src/application/training/utils/error-mapper.ts`

## No findings

Both files are internal infrastructure: discriminated union error type definitions and an HTTP status code mapper. They contain no AI writing tropes.

The only string that reaches end users is on line 82 of `training.errors.ts`:

> "User settings not found. Please complete onboarding first."

This is plain instructional text. No tropes present (no "certainly", "absolutely", "I'd be happy to", "delve", sycophantic openers, or other AI-pattern language).

All other message strings (`"Failed to generate styled text"`, `"Failed to insert training example"`, etc.) are internal error messages used for logging and developer-facing API responses, not user-visible UI copy.
