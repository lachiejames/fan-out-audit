# Audit: packages-contracts-src-config

**Files inspected**: 3
**Findings**: 3

## Summary

The config files are generally very clean. `routes.ts` and `changelog.ts` use strong `as const` patterns. `urls.ts` has two type cast usages for `import.meta` that are defensible but noteworthy. The `ChangelogEntry.date` field is a loose string rather than a structured date.

## Findings

### Finding 1: Type cast on `import.meta` in `shouldForceProdApi` and `shouldForceLocalApi`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/config/urls.ts:106`
- **Category**: type-cast
- **Impact**: low
- **Description**: Both `shouldForceProdApi` and `shouldForceLocalApi` cast `import.meta` to `{ env?: Record<string, string> }`. This is a defensive pattern for non-Vite environments, but the cast suppresses any TypeScript errors that might surface if the shape changes.
- **Suggestion**: The pattern is acceptable given the cross-environment requirement. Consider adding a comment explaining why the cast is needed (non-Vite contexts). If Zod validation is ever added to this module, validate the env value at read time.
- **Evidence**: `const meta = import.meta as { env?: Record<string, string> };` at lines 106 and 123.

### Finding 2: `ChangelogEntry.date` typed as `string` rather than a structured type

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/config/changelog.ts:8`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `date: string` in `ChangelogEntry` accepts arbitrary strings. Current values are human-readable like "April 2026", not ISO dates. There is no compile-time enforcement of this convention.
- **Suggestion**: If the format is intentionally human-readable (for display only), document it explicitly. If machine-parsing is ever needed, consider a branded string type or a `{ month: number; year: number }` shape.
- **Evidence**: `date: string;` at line 8 of `changelog.ts`.

### Finding 3: `AppRoute` and `ApiStreamRoute` type derivations filter out function members but silently accept new function entries

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/config/routes.ts:232-243`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The `AppRoute` and `ApiStreamRoute` types are derived by filtering the `const` objects to only string members. This means if a developer accidentally adds a non-function, non-string value to `appRoutes`, it would silently be included in the union. There is no constraint that ensures all non-function members are route strings.
- **Suggestion**: The current approach is reasonable. A minor improvement would be to add a `satisfies Record<string, string | ((...args: string[]) => string)>` annotation on `appRoutes` and `apiStreamRoutes` to catch non-string, non-function entries at declaration time.
- **Evidence**: `export type AppRoute = (typeof appRoutes)[keyof typeof appRoutes] extends infer R ? R extends string ? R : never : never;` at lines 232-236.
