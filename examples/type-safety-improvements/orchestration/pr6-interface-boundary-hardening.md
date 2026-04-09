# PR6: Interface Boundary Hardening

Fix the API/interface layer trust boundaries: webhook guards, SSE handlers, chat types, and controller mappers that currently rely on duplicated interfaces and inline structural casts.

## Scope

All files under `apps/api/src/interface/`.

## Violations to Fix

| Pattern                                          | Count      | Fix strategy                                          |
| ------------------------------------------------ | ---------- | ----------------------------------------------------- |
| `RawBodyRequest` duplicated in 5 webhook guards  | 5 files    | Extract one shared type and import everywhere         |
| Express header casts dropping `string[]` case    | 12+ sites  | Add shared header helper / normalized accessor        |
| SSE `req.body as { ... }` / pubsub payload casts | 7+ sites   | Parse with Zod at the controller boundary             |
| `LoggerLike` duplicated across chat module       | 4 files    | Extract to `interface/http/chat/chat-logger.types.ts` |
| BullMQ webhook job narrowing casts               | 12+ casts  | Extract shared job data types + narrowing helpers     |
| `Record<string, unknown>` mapper surfaces        | 10+ sites  | Replace with named interfaces / generics              |
| `UIMessage["parts"]` / chat inline casts         | ~4-6 sites | Use `satisfies` or shared builders instead of `as`    |

## Execution

1. Extract shared interface-layer types first:
   - `RawBodyRequest` → `apps/api/src/interface/shared/types/raw-body-request.ts`
   - `LoggerLike` → `apps/api/src/interface/http/chat/chat-logger.types.ts`
   - Webhook job data / narrowing helpers
   - Header normalization helper (see `support.controller.ts` for the correct pattern)
2. Add explicit Zod parsing to SSE endpoints and `@Res()` paths that bypass ts-rest body validation
3. Replace duplicated pub/sub payload casts with validated shared parsers
4. Tighten controller/mapper inputs that currently use `Record<string, unknown>`
5. Replace chat `UIMessage["parts"]` casts with `satisfies` or a shared typed builder
6. Run a grep sweep for `rawBody?: Buffer`, `req.body as`, and repeated header casts
7. Run `pnpm compile --filter @slopweaver/api`

## Difficulty

**Medium-high** — mostly local fixes, but shared interface-layer types must be extracted cleanly to avoid chat/webhook churn.

## Estimated Files

~18-25 files.

## Variables

- `{worktree}`: `type-safety-pr6-interface-boundaries`

## Parallel

Starts after PR3 and PR5 merge. Can run in parallel with PR8. Do not touch `apps/api/src/application/` in this PR.
