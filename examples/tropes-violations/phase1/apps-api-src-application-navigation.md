# Audit: apps-api-src-application-navigation

**Files reviewed:**

- `apps/api/src/application/navigation/errors/navigation.errors.ts`
- `apps/api/src/application/navigation/utils/error-mapper.ts`

## No findings

Both files contain only internal developer-facing code. `navigation.errors.ts` defines TypeScript error types and a factory function that produces internal error objects with a machine-readable `code` and a developer-facing `message` string (`"Failed to fetch ${source} counts"`). `error-mapper.ts` maps those error codes to HTTP status codes. Neither file contains user-facing copy, marketing language, or AI writing tropes.
