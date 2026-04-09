# Type-Safety Series

Apply the highest-impact fixes from the 2026-04-09 type-safety audit across contracts, Drizzle tables, backend abstractions, integrations, frontend app code, and the UI package. Fix root-cause weak typing first, then clean consumers, then make the most regression-prone patterns blocking.

## Precursor

The type-safety audit (`docs/audits/type-safety-improvements/`) ran 249 Phase 1 slices across 1,834 production files, followed by 21 Phase 2 cross-cutting pattern groups and a synthesis. This series turns the highest-value findings into mergeable PRs that can run in parallel without file overlap.

## PR Dependency Graph

```
PR1 ──┐
PR2 ──┤
PR3 ──┘── parallel root fixes (contracts / db tables / backend foundations)
       │
PR4 ───┬── after PR1 + PR2 + PR3 (integrations abstractions)
PR5 ───┤── after PR1 + PR2 + PR3 (application consumers)
PR7 ───┘── after PR1 + PR3 (frontend hooks / lib / pages)
       │
PR6 ───┬── after PR3 + PR5 (interface / webhooks / chat)
PR8 ───┘── after PR1 + PR7 (frontend components / shared-app / utils)
       │
PR9 ─────── after PR2 + PR4 + PR6 + PR8 (packages/ui + governance checks)
```

## Current Violation Counts (from 2026-04-09 synthesis)

| Lane                          | Approx violations                   | Biggest pattern                              |
| ----------------------------- | ----------------------------------- | -------------------------------------------- |
| Contracts schemas             | 15-20 weak metadata schemas         | `z.record(z.string(), z.unknown())`          |
|                               | 20-25 timestamp/email fields        | bare `z.string()` instead of stricter schema |
| Shared DB tables              | ~30 jsonb columns                   | missing / weak `.$type<>()`                  |
|                               | ~10 text/enum-like columns          | comment-only literal domains                 |
| Backend services + interfaces | 80+ duplicate types                 | derive-don't-duplicate violations            |
|                               | 50+ unsafe casts / weak maps        | `as T`, `Record<string, ...>`                |
| Integrations core             | 30-45 sync-service casts            | `BaseSyncService` `unknown[]`                |
|                               | ~20 metadata/cursor casts           | `platformMetadata` / cursor parsing          |
| Frontend app                  | 20+ duplicate hook/page types       | local contract drift                         |
|                               | 15+ `mutateAsync: Promise<unknown>` | return-type erasure                          |
|                               | 20+ error body casts                | `response.body as { message?: ... }`         |
| Frontend/UI maps              | 60+ finite-key maps                 | `Record<string, T>` on closed key sets       |
| **Total**                     | **200+ high-value sites**           |                                              |

## Blocking Flip Schedule

| PR  | Check                              | Action                                     |
| --- | ---------------------------------- | ------------------------------------------ |
| PR9 | `check-contract-schema-strictness` | Add after PR1 cleanup; blocking from day 1 |
| PR9 | `check-drizzle-jsonb-typing`       | Add after PR2 cleanup; blocking from day 1 |
| PR9 | `check-frontend-error-body-casts`  | Add after PR7 cleanup; blocking from day 1 |
| PR9 | `check-no-inline-ai-usage-casts`   | Add after PR3/PR4/PR5 cleanup; blocking    |

## Merge Policy

- Each PR must pass `pnpm compile --filter <affected-pkg>`
- PRs touching `packages/contracts` or `apps/api/src/shared/db/tables/` must merge before dependent consumer PRs are rebased
- Each PR must run its own grep/check sweep and show 0 remaining violations for its lane before merging
- Do not add new CLI checks in PR1–PR8; PR9 owns all governance checks and blocking flips
- PR9 lands last and runs `pnpm cli local-ci`
