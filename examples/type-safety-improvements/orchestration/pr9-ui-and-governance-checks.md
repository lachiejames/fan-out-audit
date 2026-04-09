# PR9: UI and Governance Checks

Finish the series by cleaning the remaining `packages/ui` type issues and adding blocking governance checks so the highest-value regressions cannot return.

## Scope

Files under:

- `packages/ui/src/`
- `packages/cli-tools/src/` (new checks)
- Root CLI/check registration files as needed

## Violations to Fix in packages/ui

| Pattern                                                 | Count      | Fix strategy                                                              |
| ------------------------------------------------------- | ---------- | ------------------------------------------------------------------------- |
| Polymorphic `as` prop casts duplicated                  | ~10 casts  | Extract a shared polymorphic component factory or helper type             |
| Avoidable narrowing casts in UI foundations             | ~12 casts  | Replace with type guards / better narrowing                               |
| Duplicate UI type definitions                           | 2-3 pairs  | Canonicalize (`MediaQueryFeatures`, `ColorPalette`, related shared types) |
| Organism props using `T \| null`                        | 20+ props  | Convert to optional props (`prop?: T`) with defaults                      |
| Finite-key platform maps / chart weak types             | ~5-8 sites | Tighten key unions; remove index-signature-driven casts in `chart.tsx`    |
| `return externalProps as never` in form-control-context | 1          | Refactor to proper union return type                                      |

## Checks to Create

All four checks start **blocking** (exit 1) immediately because this PR lands after the cleanup series.

### `check-contract-schema-strictness`

- Scope: `packages/contracts/src/**/*.ts`
- Enforce:
  - No bare `z.string()` on fields named `*At`, `*Date`, `*Time` — require `isoDateTimeStringSchema`
  - No bare `z.string()` on fields named `*email*` — require `z.email()`
  - No `z.record(z.string(), z.unknown())` on named metadata/payload fields where a typed schema exists
  - No duplicate `integrationPlatformSchema` / platform union definitions outside the canonical location

### `check-drizzle-jsonb-typing`

- Scope: `apps/api/src/shared/db/tables/**/*.ts`
- Enforce:
  - No bare `jsonb(...)` column without `.$type<>()`
  - Ban `.$type<Record<string, unknown>>()` in table definitions
  - Ban `.$type<unknown>()` in table definitions

### `check-frontend-error-body-casts`

- Scope: `apps/app/src/**/*.{ts,tsx}`
- Enforce:
  - Ban `response.body as { message?` and `response.body as { error?`
  - Require `extractErrorMessage` or ts-rest discriminated union narrowing

### `check-no-inline-ai-usage-casts`

- Scope: `apps/api/src/**/*.ts`
- Enforce:
  - Ban `as { usage?:` and the equivalent defensive guard+cast block around `result.usage`
  - Require direct `result.usage` via the shared typed helper from PR3

## Execution

1. Clean `packages/ui` casts and duplicate types, prioritizing the shared polymorphic abstraction
2. Tighten organism props from `T | null` to optional props with defaults
3. Fix UI finite-key record maps and chart/index-signature-driven casts
4. Fix `form-control-context.tsx` `return externalProps as never` — refactor to union return type
5. Create all four CLI checks using the standard `core.ts` / `core.test.ts` / `index.ts` pattern from existing checks
6. Register the checks in the CLI and root package wiring
7. Run each new check — must show 0 violations (cleanup series has landed)
8. Run `pnpm compile --filter @slopweaver/ui`
9. Run `pnpm cli local-ci`

## Difficulty

**Medium-high** — UI fixes are bounded, governance work is straightforward, but the final `pnpm cli local-ci` must be clean across the entire repo.

## Estimated Files

~18-28 files across `packages/ui/src/` and `packages/cli-tools/src/`.

## Variables

- `{worktree}`: `type-safety-pr9-ui-governance`

## Parallel

Must run last. Do not touch `packages/cli-tools/` in PR1–PR8; PR9 owns all new checks and blocking flips.
