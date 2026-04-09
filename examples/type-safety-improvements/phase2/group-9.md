# Cross-Cutting Patterns (Group 9)

**Slices analyzed**: apps-api-src-interface-2, apps-api-src-interface-10, apps-api-src-interface-11, apps-api-src-interface-12, apps-api-src-interface-13, apps-api-src-interface-14, apps-api-src-interface-15, apps-api-src-interface-16, apps-api-src-interface-17, apps-api-src-interface-18, apps-api-src-interface-19, apps-api-src-interface-20

## Pattern 1: Express header casts dropping `string[]` case

- **Seen in**: slice-19 (github-marketplace-webhook.controller.ts), slice-20 (all 5 webhook signature guards), slice-15 (support.controller.ts noted as positive counterexample)
- **Category**: type-cast
- **Combined impact**: Medium -- at least 7 files cast `req.headers["x-some-header"]` from `string | string[] | undefined` to `string | undefined`, silently discarding the array case. While the specific headers are single-valued by spec, the pattern bypasses the type system and is fragile if proxies concatenate or split headers.
- **What's happening**: Every webhook signature guard and the GitHub Marketplace controller extract headers with `as string | undefined`. The `support.controller.ts` file demonstrates the correct approach using `typeof` and `Array.isArray` guards, proving the codebase already has a model pattern.
- **Suggestion**: Extract a shared `getFirstHeader(headers: IncomingHttpHeaders, name: string): string | undefined` helper that normalizes `string | string[]` to `string`. Replace all direct header casts with calls to this helper. The pattern from `support.controller.ts` is the reference implementation.
- **Estimated scope**: 7+ files, 12+ cast sites across webhook guards and controllers.

## Pattern 2: Duplicate inline response object construction across CRUD endpoints

- **Seen in**: slice-14 (priority-scoring.controller.ts, push-notifications.controller.ts), slice-15 (saved-filters.controller.ts), slice-13 (memory.controller.ts)
- **Category**: duplicate-type
- **Combined impact**: Medium -- identical response object literals are constructed 2-4 times within each controller (once per CRUD endpoint). This means the same field mapping is maintained in 3-4 places per controller, across at least 4 controllers.
- **What's happening**: Controllers build the same response shape in every `.match()` callback: `list`, `getById`, `create`, `update` all manually construct `{ id, createdAt, updatedAt, ... }`. When a field is added or renamed, every occurrence must be updated in lockstep.
- **Suggestion**: Extract a `mapXToResponse(entity)` helper per controller (the pattern already exists in `proactive-response.utils.ts`). Each controller gets one mapping function that all CRUD handlers call.
- **Estimated scope**: 4+ controllers, 12-16 duplicate inline object constructions total.

## Pattern 3: `Record<string, unknown>` used where typed interfaces exist

- **Seen in**: slice-11 (integrations-controller.utils.ts, pending-integrations.controller.ts), slice-14 (proactive-response.utils.ts, proactive.controller.ts), slice-16 (sync-stream-event-payload.utils.ts, sync-stream.controller.ts), slice-2 (tool-review-edit-mapper.types.ts)
- **Category**: record-weakening
- **Combined impact**: High -- functions accept or return `Record<string, unknown>` when the actual shape is a known fixed set of keys. This forces consumers to use `as` casts on every property access, cascading unsafe casts throughout the call chain.
- **What's happening**: Utility functions like `toSafeResponse`, `toContractSettings`, `mapSyncProgressEventRowToPayload`, and `ToolReviewEditMapper` all use `Record<string, unknown>` as their input or return type. Downstream code then casts individual properties to the expected types (e.g., `payload["phase"] as SyncPhase`), compounding the unsafety.
- **Suggestion**: Replace each `Record<string, unknown>` with a named interface that reflects the actual populated keys. This eliminates all downstream casts in one shot. For `ToolReviewEditMapper`, make the type generic so callers can provide their specific input shape.
- **Estimated scope**: 6+ function signatures, with cascading elimination of 10+ downstream `as` casts.

## Pattern 4: `(error as Error).message` in catch blocks

- **Seen in**: slice-10 (database-health.indicator.ts, redis-health.indicator.ts)
- **Category**: type-cast
- **Combined impact**: Low -- only 2 files found in this group, but the pattern is likely widespread across the API layer. The cast is the standard TypeScript idiom for `unknown` catch variables, but duplicating it in every catch block is noisy and fragile (e.g., if the caught value is not an `Error`).
- **What's happening**: Both health indicators cast `error` to `Error` in catch blocks to access `.message`. This is repeated verbatim rather than using a shared utility.
- **Suggestion**: Extract `getErrorMessage(e: unknown): string` (check for `Error` instance, fallback to `String(e)`) and use it in all catch blocks. This handles non-Error thrown values safely.
- **Estimated scope**: 2 files in this group, likely 10+ across the full API layer.

## Pattern 5: DB column `string` cast to union literal types

- **Seen in**: slice-11 (sync-start-bootstrap.utils.ts -- `status as SyncRunStatus`), slice-14 (prediction-response.utils.ts -- 5 casts from DB columns to contract unions), slice-16 (sync-stream.controller.ts -- `phase as SyncPhase`), slice-18 (voice-conversation.handler.ts -- `subscriptionStatus as` inline union)
- **Category**: type-cast
- **Combined impact**: High -- at least 9 cast sites across 4 files convert DB text columns to typed union literals without runtime validation. If a DB value does not match the union (stale data, migration mismatch), the cast silently propagates an invalid value.
- **What's happening**: Drizzle text columns store enum-like values as plain strings. The interface layer casts these strings to contract union types (`SyncRunStatus`, `SyncPhase`, `PredictionIntentCategory`, `SubscriptionStatus`, etc.) without Zod validation. The `voice-conversation.handler.ts` case is especially fragile because it duplicates the union literal inline rather than importing it from contracts.
- **Suggestion**: Create Zod schemas for each union and parse DB values at the DB-to-contract boundary. A generic `parseEnum(schema, value)` helper would standardize this across all mappers. For the voice handler, import `SubscriptionStatus` from `@slopweaver/contracts` rather than inlining the union.
- **Estimated scope**: 4+ mapper/utils files, 9+ individual cast sites.

## Pattern 6: `RawBodyRequest` interface duplicated across webhook guards

- **Seen in**: slice-20 (facebook-messenger-signature.guard.ts, github-marketplace-signature.guard.ts, lemon-squeezy-signature.guard.ts, linear-signature.guard.ts, slack-signature.guard.ts)
- **Category**: duplicate-type
- **Combined impact**: High -- the exact same 3-line interface `interface RawBodyRequest extends Request { rawBody?: Buffer }` is copy-pasted in 5 files. Any change requires updating all 5 in lockstep.
- **What's happening**: Each webhook signature guard needs to access `req.rawBody` for HMAC verification. Instead of sharing a single type definition, each file declares its own local interface.
- **Suggestion**: Extract to `apps/api/src/interface/webhooks/types/raw-body-request.ts` and import in each guard. Also unify with the inline intersection variant in `webhooks-http.helpers.ts`.
- **Estimated scope**: 5 files for the interface, 1 additional file for the inline intersection variant.

## Pattern 7: `AnyTool = Tool<any, any>` propagated through agent tool system

- **Seen in**: slice-2 (ai-tool.types.ts, tool-manifest.types.ts)
- **Category**: any-usage
- **Combined impact**: Medium -- the `any` generic parameters in the `AnyTool` type alias disable type checking on tool input/output shapes throughout the entire agent tool registry. The type is declared in one file and re-declared (not imported) in another, compounding the issue.
- **What's happening**: `Tool<any, any>` is used as the common type for heterogeneous tool collections. The `tool-manifest.types.ts` file re-declares the same type inline rather than importing from `ai-tool.types.ts`, creating a maintenance risk.
- **Suggestion**: Use `Tool<z.ZodTypeAny, z.ZodTypeAny>` to retain some type constraint. Consolidate to a single declaration in `ai-tool.types.ts` and import everywhere. If TS2742 is the blocker, use targeted `@ts-expect-error` at specific call sites rather than degrading the base type.
- **Estimated scope**: 2 type declarations, but the type propagates through the entire tool registry and all tool factory functions.

## Pattern 8: `req.body as { ... }` casts bypassing Zod validation in SSE endpoints

- **Seen in**: slice-12 (knowledge-sources-upload.controller.ts), slice-16 (todo-extraction.controller.ts)
- **Category**: type-cast
- **Combined impact**: High -- SSE endpoints that use `@Res()` bypass ts-rest's automatic body validation. The controllers cast `req.body` to inline structural types without any Zod validation, meaning malformed requests will produce `undefined` field values at runtime instead of returning 400 errors.
- **What's happening**: When controllers opt into raw Express response streaming via `@Res()`, ts-rest no longer validates the request body. The controllers compensate with `req.body as { contentId: string }` style casts that provide zero runtime safety.
- **Suggestion**: Add explicit Zod schema parsing at the top of each SSE handler: `const { contentId } = bodySchema.parse(req.body)`. Return a 400 response on parse failure. This pattern should be documented as mandatory for any `@Res()` endpoint.
- **Estimated scope**: 4+ SSE endpoint handlers across 2+ controllers.

## Pattern 9: Pub/sub event payloads cast from `unknown` without validation

- **Seen in**: slice-12 (knowledge-sources-import-progress.controller.ts, knowledge-sources-import-stream.controller.ts)
- **Category**: type-cast
- **Combined impact**: Medium -- pub/sub event payloads arrive as `unknown` and are cast to typed interfaces (`ImportProgressPayload`) at multiple consumption points. If the emission shape drifts from the expected shape, the cast will silently succeed.
- **What's happening**: The pub/sub system emits events with `unknown` payloads. Multiple consumer files independently cast these to the expected payload type. The `eventType` field is also cast to an inline literal union rather than importing the type from contracts.
- **Suggestion**: Type the pub/sub emission point with Zod validation, or create a shared `parseImportProgressEvent(payload: unknown): ImportProgressPayload` helper that validates and returns the typed result.
- **Estimated scope**: 2 controller files, 3+ cast sites, plus the event type literal union duplication.
