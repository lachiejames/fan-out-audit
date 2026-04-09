# PR8: Frontend Components and Shared Types

Clean up component-level record weakening and local type drift in `apps/app` now that contracts and hook/page types are stricter.

## Scope

Files under:

- `apps/app/src/components/`
- `apps/app/src/shared-app/`
- `apps/app/src/types/`
- `apps/app/src/utils/`

## Violations to Fix

| Pattern                                      | Count       | Fix strategy                                                        |
| -------------------------------------------- | ----------- | ------------------------------------------------------------------- |
| Finite-key `Record<string, T>` maps          | 40+ maps    | Replace with union-keyed records / `as const satisfies`             |
| Local duplicate types and shared derivations | 10+ defs    | Extract one canonical shared type/constant or import from contracts |
| `platform: string` props / helpers           | 10+ sites   | Narrow to `PlatformId` or `PlatformId \| (string & {})`             |
| Post-guard `as PlatformId` casts             | ~5-10 sites | Remove after PR1 type-predicate fixes; `isPlatformId` should narrow |
| Inline struct / lookup casts from `unknown`  | 25+ sites   | Use real type guards or typed discriminated shapes                  |
| Duplicated icon/label/badge registries       | ~8 pairs    | Extract shared registries to `apps/app/src/shared-app/`             |

## High-Priority Fixes

- `getVisibleReasons()` in integrations component — 10× redundant `as Record<string, string>` casts
- `CORE_INTEGRATION_COPY`, `platformSyncStates`, `DETAIL_FIELD_LABELS`, `WORK_ITEM_TYPE_LABELS`, `TERMINAL_STATUS_BADGES` — all `Record<string, string>` on finite key sets
- `queue-audit-formatting.utils.ts`: `details: Record<string, unknown>` with cascade of `as string`/`as number` casts
- `transformTodo` in `todo-mapping.utils.ts` — large inline parameter type instead of importing `TodoResponse` from contracts
- `Window.__SLOPWEAVER_*` casts scattered across diagnostics files — consolidate into one `.d.ts` global augmentation

## Execution

1. Sweep all component/shared-app/utils finite-key maps and convert to union-keyed records or `as const satisfies`
2. Consolidate duplicated local types/constants into one canonical export per domain area
3. Tighten platform props/helpers to canonical platform type; remove redundant post-guard `as PlatformId` casts
4. Replace inline structural casts on metadata/config/lookup values with reusable type guards
5. Extract repeated icon/label/badge/platform registries into shared files
6. Consolidate `Window` global augmentations into `apps/app/src/types/globals.d.ts`
7. Run a grep sweep for `Record<string,`, `as PlatformId`, and repeated local interface names flagged by the audit
8. Run `pnpm compile --filter @slopweaver/app`

## Difficulty

**High** — many files, but almost all fixes are mechanical once PR1 and PR7 have established the canonical types.

## Estimated Files

~25-40 files.

## Variables

- `{worktree}`: `type-safety-pr8-frontend-components`

## Parallel

Starts after PR1 and PR7 merge. Can run in parallel with PR6. Must merge before PR9.
