# Tropes Audit: apps/api/src/domain/entities

**Files audited:**

- `apps/api/src/domain/entities/errors/entity.errors.ts`
- `apps/api/src/domain/entities/network/errors/network.errors.ts`
- `apps/api/src/domain/entities/network/utils/error-mapper.ts`

**Note:** The originally specified paths for `network.errors.ts` and `error-mapper.ts` did not exist. The correct paths were found under `network/errors/` and `network/utils/` respectively.

## Findings

No findings.

All three files contain only internal TypeScript code: error type definitions, discriminated union factories, and an HTTP status mapping utility. There is no user-facing copy, UI text, or hardcoded error messages surfaced to users. JSDoc comments are developer-facing documentation only.
