# PR5: Application Layer Type Alignment

Clean up the application layer now that contracts, DB tables, and shared helpers are stricter. This PR removes local type drift and weak update/cast patterns from use cases and services.

## Scope

All files under `apps/api/src/application/`.

## Violations to Fix

| Pattern                                        | Count       | Fix strategy                                                      |
| ---------------------------------------------- | ----------- | ----------------------------------------------------------------- |
| Local types duplicating contracts / DB rows    | 25+ defs    | Import or derive from contracts / `$inferSelect` / `$inferInsert` |
| `Record<string, unknown>` Drizzle `.set()`     | ~6-8 sites  | Use `Partial<typeof table.$inferInsert>` accumulators             |
| Unsafe JSONB / raw payload casts               | 20+ sites   | Parse or rely on stricter PR1/PR2 source types instead of `as T`  |
| Finite-key `Record<string, T>` maps            | 12-15 sites | Use `Record<Union, T>` / `Partial<Record<Union, T>>`              |
| `string` fields/params where unions exist      | 10-12 sites | Narrow to contract / enum types                                   |
| Copy-pasted AI usage casts in services         | ~5-6 sites  | Switch to typed helpers from PR3                                  |
| SDK type duplication / weak external responses | ~8-10 sites | Import SDK/canonical types or validate at the HTTP boundary       |

## High-Priority Fixes

- `analytics.service.ts`: `table: any`, `conditions: any[]` wipes all Drizzle type safety
- `digest.service.ts` + `digest-insights.utils.ts`: `DigestUrgentItem`/`DigestStats`/`DigestInsights` declared 3× — import from `@slopweaver/contracts`
- `triage-learning.service.ts`: `LearnedRule` is a byte-for-byte duplicate of the contracts export — delete local copy

## Execution

1. Sweep `apps/api/src/application/` for local interfaces/types that mirror contracts or DB rows; replace with imports/derivations
2. Refactor every dynamic Drizzle update payload to a typed `Partial<typeof table.$inferInsert>` object
3. Remove JSONB/raw payload casts made unnecessary by PR1/PR2; validate remaining real boundary cases
4. Tighten constant maps and lookup objects to union-keyed records
5. Replace `string` parameters/fields with the correct union/enum types
6. Delete the remaining inline AI usage-cast boilerplate and switch to PR3 helper path
7. Run a grep sweep for `Record<string, unknown>`, `.set(` accumulators, and `as { usage?:`
8. Run `pnpm compile --filter @slopweaver/api`

## Difficulty

**High** — broad application-layer cleanup with many judgment calls, but coherent once the root contracts/db/shared changes are in place.

## Estimated Files

~20-30 files.

## Variables

- `{worktree}`: `type-safety-pr5-application-consumers`

## Parallel

Starts after PR1, PR2, and PR3 merge. Can run in parallel with PR4 and PR7. Must merge before PR6.
