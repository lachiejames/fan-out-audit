# Type-Safety Audit: apps/api/src/interface (Slice 10)

**Files audited**: `health/indicators/database-health.indicator.ts`, `health/indicators/redis-health.indicator.ts`

---

## health/indicators/database-health.indicator.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

| Line | Cast                                        | Concern                                                                                                                                                                                                                                                     |
| ---- | ------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 25   | `(error as Error).message` in `catch` block | TypeScript types caught exceptions as `unknown` under `useUnknownInCatchVariables`. The cast is the standard idiom but is repeated across multiple health indicators. A shared `getErrorMessage(e: unknown): string` utility would centralise this pattern. |

### 3. `any` / Unsafe `unknown` Usage

| Location        | Usage                | Concern                                                                                 |
| --------------- | -------------------- | --------------------------------------------------------------------------------------- |
| `catch (error)` | `error` is `unknown` | Correct. The `as Error` cast is applied before accessing `.message`. Not a `any` usage. |

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## health/indicators/redis-health.indicator.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

| Line | Cast                                        | Concern                                                                                                 |
| ---- | ------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| 28   | `(error as Error).message` in `catch` block | Same pattern as `database-health.indicator.ts`. The cast is repeated rather than using a shared helper. |

### 3. `any` / Unsafe `unknown` Usage

Same note as above — the cast is safe but duplicated.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## Summary

| Severity | Finding                                                                                                        | File                                                        | Lines  |
| -------- | -------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- | ------ |
| Low      | `(error as Error).message` duplicated in both health indicators — extract `getErrorMessage(e: unknown)` helper | `database-health.indicator.ts`, `redis-health.indicator.ts` | 25, 28 |
