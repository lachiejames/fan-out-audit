# PR7: Frontend Hooks, Lib, and Pages Alignment

Fix the frontend app consumer layer: hooks, shared lib utilities, and page-level code that currently redefines contract types, erases mutation return types, and casts ts-rest error bodies.

## Scope

Files under:

- `apps/app/src/hooks/`
- `apps/app/src/lib/`
- `apps/app/src/pages/`

## Violations to Fix

| Pattern                                              | Count       | Fix strategy                                                          |
| ---------------------------------------------------- | ----------- | --------------------------------------------------------------------- |
| Local interfaces duplicating `@slopweaver/contracts` | 20+ defs    | Import / derive from contracts                                        |
| `response.body as { message?: ... }`                 | 20+ sites   | Replace with `extractErrorMessage` or proper ts-rest status narrowing |
| `mutateAsync: Promise<unknown>`                      | 15+ sites   | Propagate the actual contract-derived mutation return type            |
| `response.body as T` on successful ts-rest           | ~8 sites    | Tighten contract/body inference; use `select: (d) => d.body`          |
| Finite-key `Record<string, T>` maps                  | 20+ sites   | Use union-keyed records in hooks/lib/pages                            |
| URL/search-param and deserialization casts           | 15+ sites   | Add validators/type guards at the boundary                            |
| `platform: string` instead of `PlatformId`           | ~4-6 sites  | Narrow to canonical platform types                                    |
| Window/global/external-lib cast clusters             | ~5-10 sites | Add `.d.ts` declarations / typed wrappers where the pattern repeats   |

## High-Priority Fixes

- `BackendIntegration` defined in 3 hook files (`useIntegrationHub`, `useIntegrationsDirectory`, `usePlatformSync`) — create one canonical export and import everywhere
- `useSettingsData.ts`: `UserSettings`/`UpdateSettingsInput` local re-declarations — derive from contracts
- All hooks: replace `response.body as { message?: string | string[] }` with `extractErrorMessage` from `@slopweaver/contracts`
- `react-force-graph-2d` callback casts in `KnowledgeGraphCanvas.tsx` / `EntityGraphView.tsx` — one module augmentation or wrapper eliminates ~11 casts across both files

## Execution

1. Delete duplicate hook/lib/page types that mirror contracts and replace with imports or derived types
2. Replace all non-200 error body casts with `extractErrorMessage` or typed ts-rest narrowing
3. Propagate actual mutation return types through every hook interface that currently says `Promise<unknown>`
4. Remove successful ts-rest body casts by tightening contract/body inference at the hook boundary
5. Tighten record maps and count maps to union-keyed record types
6. Replace URL param and deserialization casts with validators/type guards
7. Add any required app-level `.d.ts` declarations for repeated window/global/library cast clusters
8. Run a grep sweep for `Promise<unknown>`, `response.body as`, and `platform: string`
9. Run `pnpm compile --filter @slopweaver/app`

## Difficulty

**High** — the biggest frontend consumer cleanup lane, but coherent and bounded to hooks/lib/pages only.

## Estimated Files

~20-30 files.

## Variables

- `{worktree}`: `type-safety-pr7-frontend-hooks-pages`

## Parallel

Starts after PR1 and PR3 merge. Can run in parallel with PR4 and PR5. Must merge before PR8 and PR9.
