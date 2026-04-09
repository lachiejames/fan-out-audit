# Audit: apps-api-src-shared-14

**Files inspected**: 8
**Findings**: 5

## Summary

The utility files in this slice are generally well-structured with `unknown`-based error handling. Key issues are: several `as Record<string, unknown>` type-cast narrowing patterns in `postgres-error.utils.ts` that could be eliminated with a proper type guard, a non-null assertion operator (`!`) used on a value that was already checked (`selectedCode!`), a duplicate `ErrorStatusCode` type defined in two separate files, and `any` usage in the `error-mapper.factory.ts` generics.

## Findings

### Finding 1: `any` in `error-mapper.factory.ts` type parameters

- **File**: `apps/api/src/shared/http/error-mapper.factory.ts:50,52,54,70`
- **Category**: any-usage
- **Impact**: medium
- **Description**: The internal implementation of `createErrorMapper` uses `any` in several generic positions: `H extends (...args: any[]) => infer R`, `Record<string, any>`, and `TCases extends Record<string, any>`. These are used in a type-only context to drive inference, but they suppress type checking on the inputs. Specifically, `createErrorMapper<TError extends Record<string, any>>()` is the entry constraint — `Record<string, unknown>` would be safer.
- **Suggestion**: Replace `Record<string, any>` with `Record<string, unknown>` where the value is never read as a specific type in the implementation body. The `any[]` in the function overload constraint is harder to eliminate but could be replaced with `unknown[]`.
- **Evidence**:
  ```ts
  type ResponseForHandler<H> = H extends ErrorStatusCode
    ? ErrorResponseForStatus<H>
    : H extends (...args: any[]) => infer R
      ? R
      : never;
  type ResponseUnion<TCases extends Record<string, any>> = { ... }
  export function createErrorMapper<TError extends Record<string, any>>(): ErrorMapperBuilder<TError>
  ```

### Finding 2: Duplicate `ErrorStatusCode` type in `error-mapper.factory.ts` and `error-response.utils.ts`

- **File**: `apps/api/src/shared/http/error-mapper.factory.ts:24` and `apps/api/src/shared/http/error-response.utils.ts:18`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: The `ErrorStatusCode = 400 | 401 | 402 | 403 | 404 | 409 | 410 | 429 | 500 | 502 | 503` union is defined identically in both files. If a new status code is added (e.g. `503` already exists, but say `422`), it must be added in two places.
- **Suggestion**: Define `ErrorStatusCode` once in `error-response.utils.ts` (where it's already exported) and import it into `error-mapper.factory.ts`.
- **Evidence**:

  ```ts
  // error-mapper.factory.ts:24
  type ErrorStatusCode = 400 | 401 | 402 | 403 | 404 | 409 | 410 | 429 | 500 | 502 | 503;

  // error-response.utils.ts:18
  type ErrorStatusCode = 400 | 401 | 402 | 403 | 404 | 409 | 410 | 429 | 500 | 502 | 503;
  ```

### Finding 3: Duplicate `StandardErrorBody` interface in `error-mapper.factory.ts` and `error-response.utils.ts`

- **File**: `apps/api/src/shared/http/error-mapper.factory.ts:28-33` and `apps/api/src/shared/http/error-response.utils.ts:27-33`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `StandardErrorBody` is defined identically in both files. The factory explicitly re-exports it "for downstream type visibility" (comment on line 27), but since `error-response.utils.ts` already exports it, the factory should simply import and re-export it rather than redefine it.
- **Suggestion**: Remove the `StandardErrorBody` definition from `error-mapper.factory.ts` and `import type { StandardErrorBody } from "@/shared/http/error-response.utils"`. If re-export is needed, use `export type { StandardErrorBody }`.
- **Evidence**:
  ```ts
  // Both files:
  export interface StandardErrorBody {
    statusCode: number;
    error: string;
    message: string[];
    code?: string;
    details?: Record<string, unknown>;
  }
  ```

### Finding 4: Non-null assertion on an already-validated value in `postgres-error.utils.ts`

- **File**: `apps/api/src/shared/db/utils/postgres-error.utils.ts:99`
- **Category**: type-cast
- **Impact**: low
- **Description**: `result.code = selectedCode!` uses a non-null assertion even though the surrounding `if (isPostgresErrorCode({ code: selectedCode }))` check already ensures `selectedCode` is defined (the function returns `false` when `code === undefined`). TypeScript does not narrow through the function boundary, requiring the `!`. This could be avoided by inlining the check.
- **Suggestion**: Refactor to inline the `undefined` check: `if (selectedCode !== undefined && isPostgresErrorCode({ code: selectedCode })) { result.code = selectedCode; }` — this removes the need for `!` entirely.
- **Evidence**:
  ```ts
  if (isPostgresErrorCode({ code: selectedCode })) {
    result.code = selectedCode!;
  }
  ```

### Finding 5: `as PostgresErrorLike` type cast in `toErrorLike` in `postgres-error.utils.ts`

- **File**: `apps/api/src/shared/db/utils/postgres-error.utils.ts:38`
- **Category**: type-cast
- **Impact**: low
- **Description**: After verifying `isObjectLike(value)`, the function casts to `PostgresErrorLike` with `value as PostgresErrorLike`. Since `PostgresErrorLike` has all-optional `unknown` fields, this cast is safe at runtime, but it is still an unsafe `as` cast. The guard `isObjectLike` narrows to `Record<string, unknown>`, which is structurally compatible.
- **Suggestion**: Change the return to `return value as unknown as PostgresErrorLike` (already acceptable for this loose interface) or, better, make `isObjectLike` return `value is PostgresErrorLike` directly since `PostgresErrorLike` is a subset of `Record<string, unknown>`.
- **Evidence**:
  ```ts
  return value as PostgresErrorLike;
  ```

## No additional findings

`work-items.table.ts`, `writing-patterns.table.ts`, `safe-query.utils.ts`, `api-call.errors.ts`, `database.errors.ts`, and `embedding.errors.ts` are clean with well-structured types and no unsafe patterns.
