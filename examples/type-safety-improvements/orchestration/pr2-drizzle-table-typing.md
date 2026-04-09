# PR2: Drizzle Table Typing

Fix the root DB typing holes so JSONB columns and enum-like text columns stop forcing downstream casts across the backend.

## Scope

All files under `apps/api/src/shared/db/tables/`.

## Violations to Fix

| Pattern                                        | Count        | Fix strategy                                                                          |
| ---------------------------------------------- | ------------ | ------------------------------------------------------------------------------------- |
| Untyped `jsonb` columns                        | ~30 columns  | Add `.$type<T>()` on every jsonb column                                               |
| `.$type<Record<string, unknown>>()`            | ~10-15 cols  | Replace with concrete interfaces or discriminated unions                              |
| Comment-only literal domains on `text()` cols  | ~10-12 cols  | Add `pgEnum` or `.$type<"a" \| "b">()`                                                |
| Platform cursor aliases all equal weak records | 15 aliases   | Replace with concrete per-platform cursor types or collapse to one honest type        |
| Weak metadata column shapes                    | ~6-8 columns | Define named metadata interfaces near the table or in `apps/api/src/shared/db/types/` |

## High-Priority Columns

Start with the highest-traffic JSONB columns:

- `predictions.predictedActions`
- `suggestions.compoundSuggestion`
- `sync-checkpoints.cursorState`
- `domain-events.payload`
- `knowledge-source-revisions` (8 untyped jsonb columns)

Gold-standard pattern: `.$type<UiPart[]>()` from `chat-messages.table.ts`.

## Execution

1. Audit all `jsonb(` columns in `apps/api/src/shared/db/tables/`
2. Add `.$type<T>()` to every untyped JSONB column starting with the high-priority list above
3. Replace `Record<string, unknown>` metadata/cursor shapes with named interfaces or discriminated unions
4. Tighten text columns with known literal domains using `pgEnum` where practical, otherwise `.$type<...>()`
5. Clean up cursor aliases so they provide real structure instead of nominal aliases over the same weak record
6. Run a grep sweep to confirm there are no bare `jsonb(...)` columns left without a meaningful `.$type<>`
7. Run `pnpm compile --filter @slopweaver/api`

## Difficulty

**High** — each table change is small, but this PR changes foundational types that ripple through large parts of the backend.

## Estimated Files

~15-20 table files.

## Variables

- `{worktree}`: `type-safety-pr2-drizzle-tables`

## Parallel

Can run in parallel with PR1 and PR3. Must merge before PR4, PR5, and PR9.
