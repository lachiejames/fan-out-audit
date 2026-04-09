# Type-Safety Audit: apps/api/src/interface (Slice 15)

**Files audited**: `refresh/refresh-all.controller.ts`, `saved-filters/saved-filters.controller.ts`, `settings/settings.controller.ts`, `suggested-actions/suggested-actions.controller.ts`, `suggestions/suggestions.controller.ts`, `summaries/summary.controller.ts`, `support/support.controller.ts`, `sync/sync-progress.controller.ts`

---

## refresh/refresh-all.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found.

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

**Assessment**: Clean file. Gold-standard pattern.

---

## saved-filters/saved-filters.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found.

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

| Location                                              | Note                                                                                                                                                                                                                                                                                                          |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `list`, `getById`, `create`, `update` response bodies | All four endpoints construct the identical inline object `{ createdAt: filter.createdAt.toISOString(), filters: filter.filters, icon: filter.icon, id: filter.id, name: filter.name, updatedAt: filter.updatedAt.toISOString() }`. A `mapFilterToResponse(filter)` helper would eliminate this 4× repetition. |

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## settings/settings.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found.

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

**Assessment**: Clean file. The model-settings policy validation is properly typed via `validateModelSettingsUpdate`.

---

## suggested-actions/suggested-actions.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found.

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

**Assessment**: Clean file. Gold-standard pattern.

---

## suggestions/suggestions.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found.

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

**Assessment**: Clean file. Gold-standard pattern.

---

## summaries/summary.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found.

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

**Assessment**: Clean file. No logger injection (no `AppLoggerService` dependency), which is a minor observability gap but not a type-safety issue.

---

## support/support.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

| Line | Cast                                                   | Concern                                                                                                                                                                                                                                                                          |
| ---- | ------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 51   | `req.headers["x-forwarded-for"]` accessed without cast | The code correctly uses `typeof forwarded === "string"` and `Array.isArray(forwarded)` type guards before accessing values. No unsafe casts. This is the correct pattern compared to `auth.controller.ts` (which casts instead of guarding). Worth noting as a positive example. |

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

**Assessment**: Clean file. `getRequestIp` correctly uses type guards rather than casts for Express header access — this is the pattern that `auth.controller.ts` should follow for its header extraction.

---

## sync/sync-progress.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found.

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

**Assessment**: Clean file. `cancelSync` discriminant check via `"notFound" in result` is a correct discriminated-union check, not an unsafe cast.

---

## Summary

| Severity | Finding                                                                                                    | File                          | Lines    |
| -------- | ---------------------------------------------------------------------------------------------------------- | ----------------------------- | -------- |
| Low      | Filter response object built identically in 4 endpoints — extract `mapFilterToResponse` helper             | `saved-filters.controller.ts` | multiple |
| Positive | `getRequestIp` uses type guards (not casts) for `x-forwarded-for` — model pattern for `auth.controller.ts` | `support.controller.ts`       | 49–59    |

All other files in this slice are clean with no type-safety issues.

---

## Cross-Slice Positive Pattern

`support/support.controller.ts` line 49–59 demonstrates the correct approach to Express header access using runtime type guards (`typeof forwarded === "string"`, `Array.isArray(forwarded)`) rather than casts. This pattern should be adopted in `auth/auth.controller.ts` where the equivalent header access currently uses `as string | undefined` casts.
