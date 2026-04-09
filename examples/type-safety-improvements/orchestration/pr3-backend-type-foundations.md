# PR3: Backend Type Foundations

Fix shared backend helpers and domain/infrastructure root abstractions that currently force copy-pasted casts across the rest of the API.

## Scope

Files under:

- `apps/api/src/shared/` (non-table files)
- `apps/api/src/domain/`
- `apps/api/src/infrastructure/` (queue/type-foundation helpers only)

## Violations to Fix

| Pattern                                            | Count        | Fix strategy                                                               |
| -------------------------------------------------- | ------------ | -------------------------------------------------------------------------- |
| Repeated `as Record<string, unknown>` error access | 15-20 sites  | Extract shared `isRecord` / typed object guards                            |
| Inline AI SDK / Voyage usage extraction casts      | 8-10 sites   | Tighten `extractInputOutputTokensFromUsage` + delete call-site boilerplate |
| Positional params in domain ports                  | ~10 methods  | Convert to named object params per `typescript-patterns.md`                |
| Bare `unknown` on domain boundary types            | ~5-6 aliases | Replace with minimal structural contracts                                  |
| Queue/module factory `any[]` and `Queue<unknown>`  | ~5-6 files   | Narrow constructors, queue generics, and exported service types            |
| Duplicate shared/domain types                      | 10+ defs     | Canonicalize shared exports (`SubscriptionStatus`, shared error types)     |

## Execution

1. Extract shared type guards/utilities first:
   - `isRecord(value: unknown): value is Record<string, unknown>`
   - typed error message/object helpers (`getErrorMessage`, `isErrorLike`)
   - typed AI usage extraction helpers (fix `extractInputOutputTokensFromUsage` to accept `LanguageModelUsage` from the `ai` package)
   - optional `isDefined<T>(value: T | null | undefined): value is T` for downstream consumers
2. Tighten `extractInputOutputTokensFromUsage` so callers use typed `result.usage` directly — delete the 8-10 inline guard+cast blocks
3. Refactor domain port method signatures from positional params to named object params
4. Replace bare `unknown` boundary aliases with the narrowest honest structural type
5. Fix queue/module factories and queue service generics so core queue lanes stop defaulting to `unknown`
6. Consolidate duplicate shared/domain type definitions to one canonical export
7. Run a grep sweep for `as { usage?:` and `as Record<string, unknown>` in the shared/domain lanes
8. Run `pnpm compile --filter @slopweaver/api`

## Difficulty

**High** — foundational refactoring. Small mistakes here multiply into noisy rebases for every later backend PR.

## Estimated Files

~18-28 files.

## Variables

- `{worktree}`: `type-safety-pr3-backend-foundations`

## Parallel

Can run in parallel with PR1 and PR2. Must merge before PR4, PR5, PR6, PR7, PR8, and PR9.
