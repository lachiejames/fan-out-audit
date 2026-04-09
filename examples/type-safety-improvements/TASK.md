# Audit Task: Type-Safety Improvements

Find type-safety improvement opportunities across the codebase. For each file batch, look for:

1. **SDK type duplication** — Custom types that duplicate SDK types (e.g. linear/sdk, googleapis, slack/web-api, @notionhq/client, @octokit/rest, etc.) — where we define our own interface/type instead of importing from the SDK
2. **Type casts** — `as SomeType` or `<SomeType>value` that could be avoided
3. **`any` / unsafe `unknown`** — usage in production code (not tests)
4. **`Record<string, ...>`** — used where a stricter const-keyed type or union would be safer
5. **Duplicate type definitions** — same shape defined in multiple places
6. **tsconfig.json settings** — missing strict flags (strictNullChecks, noUncheckedIndexedAccess, exactOptionalPropertyTypes, etc.)
7. **ESLint config** — missing @typescript-eslint rules (no-explicit-any, consistent-type-imports, no-unnecessary-type-assertion, etc.)

**Focus**: apps/api/src/, apps/app/src/, packages/contracts/src/, packages/ui/src/

**Date**: 2026-04-09
