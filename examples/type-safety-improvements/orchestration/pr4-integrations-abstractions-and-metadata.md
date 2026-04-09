# PR4: Integrations Abstractions and Metadata

Fix the shared integration root causes that force every platform lane to cast parsed items, cursor state, platform metadata, and SDK unions.

## Scope

All files under `apps/api/src/integrations/`.

## Violations to Fix

| Pattern                                        | Count          | Fix strategy                                                                  |
| ---------------------------------------------- | -------------- | ----------------------------------------------------------------------------- |
| `BaseSyncService` `parsedItems: unknown[]`     | 10-15 files    | Add generic parsed-item type parameter to the base class                      |
| `fetchThread` / `fetchOnDemandFile` untyped    | ~5-8 files     | Add per-platform result interfaces or generic return types                    |
| `platformMetadata as T` from DB `unknown`      | ~20 cast sites | Add per-platform Zod metadata schemas + shared `parsePlatformMetadata` helper |
| Cursor-state parse casts                       | ~6-7 plugins   | Add validated `parseCursorState` helper + per-platform cursor schemas         |
| OAuth/wizard `unknown` generic chains          | 2-3 core files | Replace with minimal shared structural types                                  |
| Notion/Slack/SDK union/error casts             | ~10-12 sites   | Add explicit type guards instead of inline `as`                               |
| `.filter(Boolean) as T[]` / usage-cast cleanup | ~10 sites      | Replace with typed predicates / direct typed access via PR3 helpers           |

## Key Fix: `BaseSyncService` Generic

File: `apps/api/src/integrations/core/abstractions/base-sync.service.ts`

```ts
// Before
abstract getEmbeddingTexts(parsedItems: unknown[]): string[]

// After
abstract getEmbeddingTexts(parsedItems: TParsedItem[]): string[]
// where class BaseSyncService<..., TParsedItem>
```

Each subclass parameterizes: e.g. `JiraSyncService extends BaseSyncService<..., ResolvedIssue>`.

## Execution

1. Parameterize `BaseSyncService` with the parsed-item type and propagate through all sync service subclasses
2. Add concrete return types for `fetchThread` / on-demand fetch helpers that currently return `unknown`
3. Define per-platform metadata Zod schemas and a shared `parsePlatformMetadata({ raw, schema })` helper
4. Define per-platform cursor schemas and a validated cursor parse helper
5. Tighten the OAuth base/wizard layer so it stops threading `unknown` through token/user/integration objects
6. Replace Notion/Slack inline casts with shared type guards in `apps/api/src/integrations/`
7. Sweep integration lanes for `.filter(Boolean) as T[]` and AI usage boilerplate; replace with PR3 helpers
8. Run a grep sweep for `platformMetadata as`, `parsedItems as`, and cursor cast patterns
9. Run `pnpm compile --filter @slopweaver/api`

## Difficulty

**High** — one of the most leverage-heavy PRs in the series. One base-class change eliminates 30-45 casts; the metadata pattern eliminates 20+ more.

## Estimated Files

~22-35 files.

## Variables

- `{worktree}`: `type-safety-pr4-integrations-core`

## Parallel

Starts after PR1, PR2, and PR3 merge. Can run in parallel with PR5 and PR7. Do not touch `packages/contracts` or `shared/db/tables/` in this PR.
