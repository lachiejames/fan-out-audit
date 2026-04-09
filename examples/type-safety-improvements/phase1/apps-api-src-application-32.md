# Audit: apps-api-src-application-32

**Files inspected**: 8
**Findings**: 4

## Summary

The files in this batch are generally well-typed and follow the project's discriminated union error pattern correctly. The main issues are a `Record<string, string>` where a stricter const-keyed map would be safer, a type cast (`as RefreshStep[]`) that signals a divergence between a local interface and the contract type, a structural duplicate between two error files that differ only in their module prefix, and a `"message" in error` runtime duck-type check that bypasses the discriminated union.

## Findings

### Finding 1: `CATEGORY_LABELS` typed as `Record<string, string>` instead of a stricter mapped type

- **File**: `apps/api/src/application/support/services/support.service.ts:20`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: `CATEGORY_LABELS` is declared as `Record<string, string>` even though all keys are known string literals. The lookup on line 80 (`CATEGORY_LABELS[category] ?? category`) accepts any string key silently, so a caller passing an out-of-range `category` value will always fall through to the raw string rather than a type error. The `as const` assertion on the object literal is also ineffective because the type annotation widens the inferred type back to `Record<string, string>`.
- **Suggestion**: Declare the type from the literal keys so the compiler knows the exact key set. Export a `SupportCategory` union if the service already receives a typed value, and use it as the parameter type for `category`. At minimum, remove the `Record<string, string>` annotation so `as const` takes full effect and TypeScript infers the narrow type.

```typescript
const CATEGORY_LABELS = {
  account_help: "Account help",
  bug_report: "Bug report",
  feature_request: "Feature request",
  integration_issue: "Integration issue",
  other: "Other",
} as const;

type SupportCategory = keyof typeof CATEGORY_LABELS;
```

- **Evidence**:

```typescript
const CATEGORY_LABELS: Record<string, string> = {
  account_help: "Account help",
  bug_report: "Bug report",
  feature_request: "Feature request",
  integration_issue: "Integration issue",
  other: "Other",
} as const;
```

---

### Finding 2: `steps as RefreshStep[]` cast — local interface diverges from contract type

- **File**: `apps/api/src/application/sync/refresh-all.service.ts:199`
- **Category**: type-cast
- **Impact**: medium
- **Description**: The private `RefreshStepState` interface (lines 35–41) duplicates the shape of the `RefreshStep` type exported from `@slopweaver/contracts`. The cast `steps as RefreshStep[]` on line 199 is required precisely because the compiler cannot prove they are the same type. If the contract's `RefreshStep` schema is updated (e.g., a field is renamed or made non-nullable), this cast will silently produce a runtime mismatch without a compile error.
- **Suggestion**: Remove `RefreshStepState` and use `RefreshStep` (imported from `@slopweaver/contracts`) directly as the element type for the `steps` array. This eliminates the cast and ensures the mutable step objects are always structurally compatible with the response type.
- **Evidence**:

```typescript
interface RefreshStepState {
  name: string;
  status: RefreshStepStatus;
  startedAt: string | null;
  completedAt: string | null;
  error: string | null;
}
// ...
steps: steps as RefreshStep[],
```

---

### Finding 3: Structural duplicate — `ContextStatusErrors` and `SystemStatusErrors` are identical

- **File**: `apps/api/src/application/system/context-status/errors/context-status.errors.ts` and `apps/api/src/application/system-status/errors/system-status.errors.ts`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: Both files define:
  - A `const ErrorCode` object with a single `INTERNAL_ERROR` key
  - A type alias of the same name that is `(typeof ErrorCode)[keyof typeof ErrorCode]`
  - A `BaseXxxError` interface with `{ code: XxxErrorCode; message: string }`
  - A `XxxInternalError` interface that extends the base and adds `cause?: unknown`
  - A union type alias that is just the single `InternalError`
  - A factory object with a single `internal(message, cause?)` function

  The only difference between the two files is the type-name prefix (`ContextStatus` vs `SystemStatus`). Any future changes (e.g., adding a `NOT_FOUND` code) must be made in both files.

- **Suggestion**: Extract a shared `createInternalErrorModule` generic factory, or simply share a common `InternalError` base type in `shared/errors/` and re-export with a module-specific alias. Alternatively, if these two modules will always have the same error surface, consider whether they need to be separate error modules at all.
- **Evidence**:

```typescript
// context-status.errors.ts
export const ContextStatusErrorCode = { INTERNAL_ERROR: "INTERNAL_ERROR" } as const;
export interface ContextStatusInternalError extends BaseContextStatusError {
  code: typeof ContextStatusErrorCode.INTERNAL_ERROR;
  cause?: unknown;
}
export type ContextStatusError = ContextStatusInternalError;

// system-status.errors.ts  (identical structure, different prefix)
export const SystemStatusErrorCode = { INTERNAL_ERROR: "INTERNAL_ERROR" } as const;
export interface SystemStatusInternalError extends BaseSystemStatusError {
  code: typeof SystemStatusErrorCode.INTERNAL_ERROR;
  cause?: unknown;
}
export type SystemStatusError = SystemStatusInternalError;
```

---

### Finding 4: Unsafe duck-type check on error union bypasses discriminated union

- **File**: `apps/api/src/application/sync/refresh-all.service.ts:147`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: Inside the Step 4 error handler the code checks `"message" in error` before accessing `error.message`, then falls back to a `type` field check (`error.type === "not_found"`). This runtime duck-typing is a sign that the error type returned by `coreProfileService.refreshProfile` is not fully known at the call site, or that it is a union where not all variants share `message`. The compiler allows this only because the condition is applied to `result.error` of type `unknown` or a wide union. If the error type for `CoreProfileError` is a discriminated union, the correct pattern is an exhaustive `switch (error.code)`.
- **Suggestion**: Import the `CoreProfileError` type and use an exhaustive switch on `error.code`. If the error union genuinely has variants without `message`, add `message` to those variants or create a shared `errorToMessage` helper.
- **Evidence**:

```typescript
const error = result.error;
const message = "message" in error ? error.message : error.type === "not_found" ? "Profile not found" : "Profile error";
throw new Error(message);
```

## No additional findings

The remaining files (`summary.errors.ts`, `summary.service.ts`, `support-errors.ts`, `system-status.service.ts`) are clean: no `any`, no unsafe `unknown` casts, no SDK type duplication, and explicit return types throughout.
