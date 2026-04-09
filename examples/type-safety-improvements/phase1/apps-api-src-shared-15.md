# Audit: apps-api-src-shared-15

**Files inspected**: 8
**Findings**: 3

## Summary

This slice is mostly clean. The error types follow good discriminated-union patterns. The notable issues are: a `DatabaseError` interface in `message.errors.ts` that shares a name with the `DatabaseError` in `database.errors.ts` (creating an accidental shadow), the `AuditSuspiciousActivity.severity` field in `ai.types.ts` using `"low" | "medium" | "high" | string` (a widened union that degrades to `string`), and `DomainEvent.toOutboxPayload()` casting `this` to `Record<string, unknown>` which is an unsafe spread.

## Findings

### Finding 1: `severity` field typed as `"low" | "medium" | "high" | string` (union collapse)

- **File**: `apps/api/src/shared/types/ai.types.ts:12`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `AuditSuspiciousActivity.severity` is typed as `"low" | "medium" | "high" | string`. Adding a `string` member to a union with literal types collapses the union to `string` — the literals provide no compile-time narrowing and editor completions. Any string value is accepted.
- **Suggestion**: Remove `| string` to keep the type as `"low" | "medium" | "high"`. If extensibility is needed for future severity levels, add them explicitly to the union rather than widening to `string`.
- **Evidence**:
  ```ts
  export type AuditSuspiciousActivity = {
    severity: "low" | "medium" | "high" | string;
  };
  ```

### Finding 2: `DatabaseError` in `message.errors.ts` shadows the shared `DatabaseError` from `database.errors.ts`

- **File**: `apps/api/src/shared/errors/message.errors.ts:144-149`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `message.errors.ts` exports a `DatabaseError` interface (with `code: "DATABASE_ERROR"` and `operation: string`) that is a member of the `MessageError` union. The name `DatabaseError` also refers to the shared infrastructure type in `shared/errors/database.errors.ts` (PostgreSQL error shape). Any file that imports from both locations must use qualified names or aliases — a readability hazard and potential for silent import resolution errors.
- **Suggestion**: Rename the `MessageError` database variant to something more specific such as `MessageDatabaseError` (with `code: "MESSAGE_DATABASE_ERROR"`) to avoid the naming collision. Update the union type and factory accordingly.
- **Evidence**:
  ```ts
  // message.errors.ts
  export interface DatabaseError extends BaseMessageError {
    readonly code: "DATABASE_ERROR";
    readonly operation: string;
    readonly originalError?: string;
  }
  // database.errors.ts
  export interface DatabaseError {
    readonly message: string;
    readonly cause?: unknown;
    readonly code?: string;
    // ...
  }
  ```

### Finding 3: `as Record<string, unknown>` cast of `this` in `DomainEvent.toOutboxPayload()`

- **File**: `apps/api/src/shared/events/base.event.ts:27`
- **Category**: type-cast
- **Impact**: low
- **Description**: `toOutboxPayload` spreads `this` by casting it: `const payload: Record<string, unknown> = { ...(this as Record<string, unknown>) }`. The `as` cast is required because TypeScript does not allow spreading `this` from an abstract class directly. While safe at runtime (all class fields are enumerable), it is an unsafe cast.
- **Suggestion**: Use `Object.assign({} as Record<string, unknown>, this)` or `{ ...Object.fromEntries(Object.entries(this)) }` to avoid the cast. Alternatively, if subclasses always implement a typed `toPayload()` method, delegate to that instead of generic spreading.
- **Evidence**:
  ```ts
  const payload: Record<string, unknown> = { ...(this as Record<string, unknown>) };
  ```

## No additional findings

`message.errors.ts` (aside from the naming collision), `error-mapper.factory.ts` (covered in slice 14), `error-response.utils.ts`, `build-content-identifiers.ts`, `system-status.utils.ts`, and `billing.types.ts` are otherwise clean.
