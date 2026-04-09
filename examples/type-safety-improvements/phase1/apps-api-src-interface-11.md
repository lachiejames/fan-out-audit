# Type-Safety Audit: apps/api/src/interface (Slice 11)

**Files audited**: `integrations/integration-lifecycle.controller.ts`, `integrations/integration-sync.controller.ts`, `integrations/integrations-controller.utils.ts`, `integrations/pending-integrations.controller.ts`, `integrations/sync-start-bootstrap.utils.ts`, `integrations/unified-integrations.controller.ts`

---

## integrations/integration-lifecycle.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

| Line | Cast                                           | Concern                                                                                                                                                                                                                                                                                |
| ---- | ---------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 298  | `body.syncSettings as Record<string, unknown>` | The contract body type for this field is a typed object, but the controller casts it to a permissive record before passing to the service. If the service accepts `Record<string, unknown>`, the service signature should be updated to accept the typed shape, eliminating this cast. |

### 3. `any` / Unsafe `unknown` Usage

| Location      | Usage                                                  | Concern                                                                                                                                                       |
| ------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Lines 108–125 | Large inline anonymous type for `items` array elements | Not `any`, but the inline anonymous type is long. Extracting it as a named type (e.g. `type IntegrationListItem`) would improve readability and allow re-use. |

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

| Line | Usage                                          | Concern                                                                          |
| ---- | ---------------------------------------------- | -------------------------------------------------------------------------------- |
| 298  | `body.syncSettings as Record<string, unknown>` | See above. The body already has a typed contract shape; the cast is unnecessary. |

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## integrations/integration-sync.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

| Line | Cast                                                                                     | Concern                                                                                                                                                                                                                                                                                             |
| ---- | ---------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 78   | `(req as AuthenticatedRequest & { budgetExceeded?: BudgetExceededInfo }).budgetExceeded` | The budget guard attaches `budgetExceeded` to the request object at runtime. TypeScript has no way to know this without the intersection cast. The cleaner approach is to define a `BudgetAwareRequest` type that extends `AuthenticatedRequest` and use it as the parameter type for this handler. |

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## integrations/integrations-controller.utils.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found in this file, but the `toSafeResponse` function takes `integration: Record<string, unknown>`. See finding below.

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

| Location                   | Usage                                  | Concern                                                                                                                                                                                                                                                                                                        |
| -------------------------- | -------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `toSafeResponse` parameter | `integration: Record<string, unknown>` | The function strips sensitive token fields from an integration record. Using `Record<string, unknown>` hides the actual shape. A typed integration interface (omitting `oauthTokens` / `encryptedToken`) would make the stripping logic statically verifiable and prevent regressions when fields are renamed. |

### 5. Duplicate Type Definitions

| Location         | Note                                                                                                                                                                                        |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `toSafeResponse` | A second, independent `toSafeResponse` function exists in `pending-integrations.controller.ts`. Both strip tokens from integration records. This is duplicate logic that should be unified. |

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## integrations/pending-integrations.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

| Line(s)           | Cast                         | Concern                                                                                                                                                                          |
| ----------------- | ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 49                | `as SafeIntegrationResponse` | After `toSafeResponse` builds the object, TypeScript cannot confirm it satisfies the named type. Use `satisfies SafeIntegrationResponse` to get a structural compile-time check. |
| 95, 99, 104, etc. | `as ErrorBody`               | Repeated casts to `ErrorBody` after `.match()` callbacks. If `mapErrorToResponse` returns `ErrorBody` directly, this cast would be redundant.                                    |

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

| Line  | Usage                                                          | Concern                                                                                                                                                                                                |
| ----- | -------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 28–29 | `type IntegrationWithTokens = Record<string, unknown> & {...}` | This local type is a permissive intersection. The `Record<string, unknown>` base allows any extra fields. A precise `IntegrationWithTokens` interface that explicitly lists all fields would be safer. |

### 5. Duplicate Type Definitions

| Location                  | Note                                                                                          |
| ------------------------- | --------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| Lines 36–42               | Local `ErrorBody` interface                                                                   | This may duplicate an error-body type from `shared/http/`. If a shared type exists, import it. |
| `toSafeResponse` function | Duplicate of the same function in `integrations-controller.utils.ts`. Should be consolidated. |

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## integrations/sync-start-bootstrap.utils.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

| Line | Cast                                  | Concern                                                                                                                                                                                                                                                      |
| ---- | ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 98   | `existingRun.status as SyncRunStatus` | The `status` column is a `text` DB column (per the database patterns rule: no DB enums for platforms/statuses). After reading from Drizzle, the type is `string`. A Zod parse against the enum schema would narrow correctly and surface runtime violations. |

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## integrations/unified-integrations.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

| Lines   | Cast                                                                               | Concern                                                                                                                                                                                                                                                                             |
| ------- | ---------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 376–386 | `Reflect.get(body as object, "content")` and `Reflect.get(body as object, "text")` | These casts are used to safely access optional keys on an unknown reply body. An alternative with better type safety is optional chaining on a typed interface: `(body as { content?: string }).content` — or a Zod schema for the reply body shape that validates at the boundary. |

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## Summary

| Severity | Finding                                                                                | File                                                                     | Lines           |
| -------- | -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ | --------------- |
| High     | Dual `toSafeResponse` implementations across two files — consolidate into one          | `integrations-controller.utils.ts`, `pending-integrations.controller.ts` | —               |
| Medium   | `integration: Record<string, unknown>` in `toSafeResponse` — use typed interface       | `integrations-controller.utils.ts`                                       | param signature |
| Medium   | `req as AuthenticatedRequest & { budgetExceeded? }` — define `BudgetAwareRequest` type | `integration-sync.controller.ts`                                         | 78              |
| Medium   | `body.syncSettings as Record<string, unknown>` — use typed contract shape              | `integration-lifecycle.controller.ts`                                    | 298             |
| Medium   | `existingRun.status as SyncRunStatus` — use Zod parse for runtime validation           | `sync-start-bootstrap.utils.ts`                                          | 98              |
| Medium   | `Reflect.get(body as object, ...)` — use typed optional chaining or Zod schema         | `unified-integrations.controller.ts`                                     | 376–386         |
| Low      | `IntegrationWithTokens = Record<string, unknown> & {...}` — use precise interface      | `pending-integrations.controller.ts`                                     | 28–29           |
| Low      | Repeated `as ErrorBody` casts — if mapper returns `ErrorBody`, casts are redundant     | `pending-integrations.controller.ts`                                     | 95, 99, 104+    |
| Low      | `as SafeIntegrationResponse` should use `satisfies` for compile-time check             | `pending-integrations.controller.ts`                                     | 49              |
| Low      | Large inline anonymous type for `items` — extract as `IntegrationListItem`             | `integration-lifecycle.controller.ts`                                    | 108–125         |
