# Audit: packages-contracts-src-validation-helpers.ts

**Files inspected**: 1
**Findings**: 2

## Summary

`validation-helpers.ts` is a clean utility file with pure validation functions for use in Zod `.refine()` calls and standalone date/object checks. All functions follow the named-params convention. The file is well-documented with JSDoc. The main concerns are minor: `hasAtLeastOneField` accepts `Record<string, unknown>` which is overly permissive for its purpose, and `CustomDurationData` is an internal interface that leaks into the module boundary via the exported `requireCustomTimeWhenCustomDuration` function's parameter type.

## Findings

### Finding 1: `hasAtLeastOneField` parameter type `Record<string, unknown>` is too broad

- **File**: `packages/contracts/src/validation-helpers.ts:43`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `hasAtLeastOneField(data: Record<string, unknown>): boolean` accepts `Record<string, unknown>`. This is used as a `.refine()` predicate for update schemas. The function works correctly but allows any record, including arrays and other non-plain-object types that happen to have string keys. For a schema-validation helper, the type could be narrowed to indicate it expects an object.
- **Suggestion**: The change is cosmetic given `.refine()` usage, but for correctness the parameter could be typed as `object` (or kept as `Record<string, unknown>`). Add a JSDoc note clarifying that arrays will return `true` if they have indices (since `Object.keys([1,2,3])` returns `["0","1","2"]`).
- **Evidence**: `export function hasAtLeastOneField(data: Record<string, unknown>): boolean {` at line 43.

### Finding 2: `CustomDurationData` interface is private but enables the exported function signature

- **File**: `packages/contracts/src/validation-helpers.ts:135`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `interface CustomDurationData` is declared without `export` but is used as the parameter type of `requireCustomTimeWhenCustomDuration({ data }: { data: CustomDurationData })`. Since `CustomDurationData` is unexported, callers cannot explicitly import and use this interface to type their objects. They must rely on structural compatibility.
- **Suggestion**: Either export `CustomDurationData` so callers can explicitly annotate their snooze/schedule objects, or inline the type in the function parameter: `{ data: { duration: string; customTime?: string | Date } }`. The inline approach avoids polluting the module's public API with an internal interface.
- **Evidence**: `interface CustomDurationData { customTime?: string | Date; duration: string; }` at line 135 — no `export` keyword.
