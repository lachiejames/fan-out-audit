# Type-Safety Audit: apps/api/src/interface (Slice 6)

**Files audited**: `auth/auth.controller.ts`, `billing/billing-controller.utils.ts`

---

## auth/auth.controller.ts

### 1. Custom Types Duplicating SDK Types

No custom types that duplicate SDK types found.

### 2. Type Casts That Could Be Avoided

Multiple casts are required because Express types headers as `string | string[] | undefined`, not narrowed per-header. These are unavoidable given the Express type system, but could be wrapped in named helper functions to make them self-documenting.

| Line(s)  | Cast                                                                                           | Reason / Avoidability                                                                                                                            |
| -------- | ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| 64, 139  | `req.headers["x-platform"] as string \| undefined`                                             | Express types all headers as `string \| string[] \| undefined`. Unavoidable without a typed header-extraction helper.                            |
| 73, 148  | `req.headers["cf-connecting-ip"] as string \| undefined`                                       | Same Express header limitation.                                                                                                                  |
| 487, 598 | `req.cookies as Record<string, string \| undefined> \| undefined`                              | `req.cookies` is typed as `any` by `@types/express`; cast to `Record` is the safest narrowing available.                                         |
| 495, 605 | `platform as "web" \| "tauri-desktop" \| "tauri-mobile"`                                       | After Zod validation, TypeScript does not narrow string to the union literal. Could use `z.infer<typeof schema>` instead of casting after parse. |
| 409      | `req.app.get("CSRF_GENERATE_TOKEN") as ((req: Request, res: Response) => string) \| undefined` | `app.get()` returns `any`. Cast is required. Could be replaced with a typed module-level constant rather than storing on `app`.                  |

**Recommendation**: Extract header-extraction logic into a `getStringHeader({ req, name })` helper that encapsulates the cast once. The `platform` cast after Zod `.safeParse` could be eliminated by using `parsed.data` (already narrowed to the union) instead of re-casting the original variable.

### 3. `any` / Unsafe `unknown` Usage

No raw `any` in production code found. `req.app.get()` returns `any` implicitly but is immediately cast.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

| Line(s)  | Usage                                                             | Concern                                                                                                           |
| -------- | ----------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| 487, 598 | `req.cookies as Record<string, string \| undefined> \| undefined` | Permissive but correct: cookies are always string values. No stricter shape is known at compile time. Acceptable. |

### 5. Duplicate Type Definitions

No duplicate type definitions within this file.

### 6. Missing Strict tsconfig/eslint Flags

No file-level overrides found. Follows project-wide config.

---

## billing/billing-controller.utils.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found.

### 3. `any` / Unsafe `unknown` Usage

| Location                                    | Usage                               | Concern                                                                                                                  |
| ------------------------------------------- | ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `extra?: Record<string, unknown>` parameter | Used as structured logging metadata | Acceptable: this is an open-ended log bag where callers pass arbitrary keys. `unknown` is the correct element type here. |

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

`extra?: Record<string, unknown>` is intentionally permissive for the logging use case.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## Summary

| Severity | Finding                                                                 | File                 | Lines                                |
| -------- | ----------------------------------------------------------------------- | -------------------- | ------------------------------------ |
| Medium   | 5 header/cookie casts that could be centralised in helpers              | `auth.controller.ts` | 64, 73, 139, 148, 487, 495, 598, 605 |
| Low      | `platform` cast after Zod parse redundant if using `parsed.data`        | `auth.controller.ts` | 495, 605                             |
| Low      | `req.app.get("CSRF_GENERATE_TOKEN")` cast avoidable with typed accessor | `auth.controller.ts` | 409                                  |
