# PR1: Contract Schema Hardening

Fix the highest-leverage contract issues first so downstream API, app, and DB consumers can import stricter canonical types instead of widening locally.

## Scope

All files under `packages/contracts/src/`.

## Violations to Fix

| Pattern                                | Count        | Fix strategy                                                                   |
| -------------------------------------- | ------------ | ------------------------------------------------------------------------------ |
| `z.record(z.string(), z.unknown())`    | 15-20 fields | Replace with explicit object schemas or discriminated unions                   |
| Bare `z.string()` timestamps           | 20-25 fields | Replace with `isoDateTimeStringSchema`                                         |
| Bare `z.string()` emails               | 6-8 fields   | Replace with `z.email()`                                                       |
| Platform fields as raw `z.string()`    | 8-12 fields  | Consolidate on canonical platform schema / `PlatformId`-aware typing           |
| Duplicate schemas/types across modules | 8+ pairs     | Pick one canonical schema source and re-export / derive everywhere             |
| Weak / missing type predicates         | 2-4 helpers  | Make guards like `isPlatformId` proper type predicates (`value is PlatformId`) |

## Execution

1. Audit `packages/contracts/src/` for `z.record(z.string(), z.unknown())`, bare timestamp/email/platform fields, and duplicate schema pairs
2. Replace the highest-traffic weak metadata/payload schemas first:
   - `integrationActionPayloadSchema`
   - `baseContentSchema.metadata`
   - `baseWorkItemSchema.aiSources`
3. Bulk-replace timestamp fields with `isoDateTimeStringSchema`
4. Bulk-replace email fields with `z.email()`
5. Consolidate duplicate schemas/types so schemas own the Zod definition and types re-export `z.infer<>`
6. Fix `isPlatformId` and any related platform guards to return proper type predicates
7. Run a grep sweep to confirm the targeted weak schema patterns are gone from the contracts lane
8. Run `pnpm compile --filter @slopweaver/contracts`

## Difficulty

**High** — contract fixes are mechanically straightforward but have broad downstream blast radius; every change must preserve the public API surface intentionally.

## Estimated Files

~15-22 files in `packages/contracts/src/`.

## Variables

- `{worktree}`: `type-safety-pr1-contracts`

## Parallel

Can run in parallel with PR2 and PR3. Must merge before PR4, PR5, PR7, PR8, and PR9.
