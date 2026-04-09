# Cross-Cutting Patterns (Group 2)

**Slices analyzed**: apps-api-src-application-3, apps-api-src-application-20, apps-api-src-application-21, apps-api-src-application-22, apps-api-src-application-23, apps-api-src-application-24, apps-api-src-application-25, apps-api-src-application-26, apps-api-src-application-27, apps-api-src-application-28, apps-api-src-application-29, apps-api-src-application-30

## Pattern 1: Hand-rolled interfaces that duplicate contract/Zod-inferred types

- **Seen in**: slices 20, 21, 22, 23, 24, 25, 28, 29
- **Category**: duplicate-type
- **Combined impact**: high -- this is the single most pervasive type-safety gap in the audited code. When contract schemas evolve, these local copies silently drift, creating invisible mismatches between what the API returns and what the contract promises.
- **What's happening**: Services and utilities define local `interface` or `type` blocks that mirror shapes already exported from `@slopweaver/contracts` (Zod-inferred) or from Drizzle table `$inferSelect`/`$inferInsert` types. Examples include `PersonaResponse` (slice 24), `NormalizedPredictionOutput` (slice 25), `NetworkNode` (slice 23), `MemoryLearning` (slice 23), `ExtractedKnowledgeItem` (slice 21), `PushNotificationHistoryItem` (slice 29), `MediaFileDTO` (slice 22, defined three times), `ProactiveTriggerType`/`ProactiveUrgencyLevel` (slice 28), and `PredictionFeedbackRecord.explicitFeedback` (slice 24). In several cases the local copy uses wider types (e.g., `string` instead of a literal union), meaning callers lose exhaustiveness checking.
- **Suggestion**: Adopt a "derive, don't duplicate" rule. Every service-layer type that represents a contract response or DB row should be derived from the canonical source (`z.infer<typeof schema>`, `ServerInferResponses<typeof contract.X, 200>["body"]`, or `typeof table.$inferSelect`). Create a lint rule or code review checklist item: "Does this `interface`/`type` duplicate an existing contract or table type?"
- **Estimated scope**: 15-20 files, ~25 distinct type definitions to replace with imports or derivations.

## Pattern 2: `Record<string, unknown>` used as Drizzle `.set()` update payload

- **Seen in**: slices 20, 24, 27, 30
- **Category**: record-weakening
- **Combined impact**: high -- bypasses Drizzle's compile-time column-name and value-type checking on every update path that uses this pattern. A misspelled column name or wrong value type compiles silently and fails only at runtime.
- **What's happening**: Multiple services build a mutable `Record<string, unknown>` object (or use a conditional-type trick that collapses to `Record<string, unknown>`), dynamically assign keys via bracket notation, then pass it to Drizzle's `.set()`. This pattern appears in `knowledge-sources.service.ts` (slice 20, double-cast through conditional type), `persona.service.ts` (slice 24), `morning-brief.service.ts` / `proactive-observation.service.ts` (slice 27), and `settings.service.ts` (slice 30). In each case, Drizzle's `$inferInsert` partial type is available but not used.
- **Suggestion**: Replace all `Record<string, unknown>` update accumulators with `Partial<typeof table.$inferInsert>`. Build the object with named property assignments (not bracket notation). If conditional spreading is needed, use `...( condition && { columnName: value })` with the typed partial object. This gives column-name autocomplete, value-type checking, and catches renames at compile time.
- **Estimated scope**: 4-6 files, each with one update method to refactor.

## Pattern 3: Inline string-literal unions that duplicate DB enums or contract Zod enums

- **Seen in**: slices 20, 21, 24, 28, 29, 30
- **Category**: duplicate-type
- **Combined impact**: high -- if any enum gains a variant (e.g., a new `ArchivePhase`, a new `PredictedAction`, a new `PushChannel`), the hand-written union silently falls out of sync with the DB/contract source of truth.
- **What's happening**: Parameters and interface fields are typed with hand-written string-literal unions that are byte-for-byte identical to enums already exported from `enums.ts` (Drizzle pgEnum) or `@slopweaver/contracts` (Zod enum). Examples: `ArchiveStatus`/`ArchivePhase` (slice 20), `NeighborCategory` duplicating `KnowledgeCategory` (slice 21), `ExplicitFeedback` (slice 24), `ProactiveTriggerType`/`ProactiveUrgencyLevel` (slice 28), `PushChannel`/`PushUrgencyTier`/`PushDeliveryStatus` (slice 29), `dimensionType` (slice 30), `ArchiveSelectionResult.disposition` (slice 20).
- **Suggestion**: For every inline string-literal union, check whether an equivalent type exists in `@slopweaver/contracts` or `@/shared/db/tables/enums.ts`. If so, import and use it. If not, create the canonical type in the appropriate location and import it. Consider an ESLint rule or grep-based CI check that flags inline unions longer than 3 variants that match known enum patterns.
- **Estimated scope**: 10-15 files, ~15 inline unions to replace with imports.

## Pattern 4: Unsafe `as T` casts on JSONB columns and raw SQL rows without runtime validation

- **Seen in**: slices 20, 21, 24, 26, 27, 29
- **Category**: type-cast
- **Combined impact**: high -- JSONB columns return `unknown` from Drizzle. Bare `as T` casts assert a shape without verifying it at runtime. If stored data is from an older schema version, is corrupt, or has an unexpected shape, the cast silently succeeds and propagates wrong data to clients.
- **What's happening**: Response mappers and service methods cast JSONB column values directly to contract types (`as KnowledgeSourceMetrics | null`, `as CompoundSuggestion | null`, `as Record<string, unknown>`, etc.) without any Zod parse or type-guard validation. Raw SQL query results (`execute()` returning `Record<string, unknown>` rows) are cast field-by-field with `as string`, `as NeighborCategory`, etc. (slice 21). The same pattern appears in `knowledge-source-response.mappers.ts` (slice 20, 8+ JSONB fields), `confidence-calibration.service.ts` (slice 24), `morning-brief-context.service.ts` (slice 26, `as ResolvedEntity`), `morning-brief.service.ts` (slice 27, `as number` on `violation.details`), `fcm-push.service.ts` (slice 29), and `apns-push.service.ts` (slice 29).
- **Suggestion**: Introduce a shared `parseJsonb<T>({ value, schema }: { value: unknown; schema: z.ZodType<T> }): T | null` utility that wraps `schema.safeParse(value)`. Use it at every JSONB extraction boundary. For raw SQL queries, either migrate to typed Drizzle `select()` calls or validate rows with a Zod schema before mapping. For external API responses (APNs, FCM), use a minimal Zod schema instead of `as` casts.
- **Estimated scope**: 10-12 files, ~30 individual `as T` casts on JSONB/raw-SQL values.

## Pattern 5: `Record<string, number | string>` where `Record<PlatformId, ...>` or a const-keyed type would be stricter

- **Seen in**: slices 21, 26, 27, 28
- **Category**: record-weakening
- **Combined impact**: medium -- allows arbitrary string keys where only a known set of platform IDs or provider names are valid. Typos in keys compile silently. Fallback patterns (`?? default`) mask the fact that the Record type claims every key has a value.
- **What's happening**: Several config maps and accumulator objects use `Record<string, T>` where the valid keys are a known finite set: `PROVIDER_CONFIGS` maps known provider literals (slice 21), `unreadByPlatform` maps platform IDs (slice 26), `platformLabelMap` maps three platform strings (slice 28), `aggregatePlatformCounts` returns `Record<string, number>` (slice 28), `metadata` observation payload (slice 27). In each case, a `PlatformId`, `KnowledgeSourceProvider`, or other literal union from contracts could constrain the keys.
- **Suggestion**: Import the canonical key union from `@slopweaver/contracts` and use `Partial<Record<KeyUnion, T>>` (partial because not all keys may be present) or use `satisfies` to validate the object literal against the constrained record type. For accumulator patterns like `reduce`, supply the generic type parameter to `reduce<Partial<Record<PlatformId, number>>>` instead of casting the initial `{}`.
- **Estimated scope**: 6-8 files, ~8 Record types to tighten.

## Pattern 6: `platform: string` parameters where `PlatformId` from contracts is available

- **Seen in**: slices 3, 25, 28
- **Category**: missing-strict-typing
- **Combined impact**: medium -- multiple service methods accept `platform: string` even though `PlatformId` (a union of all valid platform literal strings) is exported from `@slopweaver/contracts`. A typo like `"githb"` compiles silently.
- **What's happening**: Internal helper functions (`resolveIntegration`, `PlatformActionsUseCase.resolveIntegrationId`, `GenerateContextualEmbeddingParams`, `calculateMessageType`, `getPlatformLabel`) accept `platform: string`. These values ultimately flow to integration lookup ports that expect a valid platform ID. The wide `string` type means the compile-time check is deferred to runtime.
- **Suggestion**: Change `platform: string` to `platform: PlatformId` (imported from `@slopweaver/contracts`) at every internal service/helper boundary. Keep `string` only at the external API controller boundary where the contract validates via Zod.
- **Estimated scope**: 6-10 files, ~10 parameters to narrow.

## Pattern 7: Redundant defensive `typeof result === "object"` guards and `as { usage?: unknown }` casts on typed AI SDK results

- **Seen in**: slices 21, 27
- **Category**: type-cast
- **Combined impact**: low -- the casts are unnecessary (the AI SDK's `generateText` already returns a fully-typed object with `.usage`), but they are not unsafe. They add noise and throw away the SDK's precise type.
- **What's happening**: After calling `generateText` from the Vercel AI SDK, the code defensively checks `typeof result === "object" && result !== null && "usage" in result`, then casts to `{ usage?: unknown }` before extracting token counts. This pattern appears identically in `knowledge-extraction.service.ts` (slice 21) and `morning-brief.service.ts` (slice 27). The defensive check is dead code since `result` is already typed.
- **Suggestion**: Remove the defensive guard and cast. Access `result.usage` directly. If `extractInputOutputTokensFromUsage` needs `unknown` for multi-SDK compatibility, pass `result.usage` without the intermediate cast. Fix in both files simultaneously since the pattern is identical.
- **Estimated scope**: 2 files, identical 3-line blocks to simplify.

## Pattern 8: `{} as Record<K, number>` empty-object casts that produce incorrect runtime values

- **Seen in**: slices 21, 28
- **Category**: type-cast
- **Combined impact**: medium -- tells TypeScript an empty object has all required keys populated, but at runtime accessing any key yields `undefined`, not `0`. Downstream arithmetic silently produces `NaN`.
- **What's happening**: Early-return paths in `knowledge-extraction.service.ts` (slice 21) return `{ itemsByCategory: {} as Record<KnowledgeCategory, number>, itemsExtracted: 0 }` -- the happy path correctly initializes all six category keys to `0`, but error/early-exit paths use the cast. Similarly, `aggregatePlatformCounts` (slice 28) seeds a `reduce` accumulator with `{} as Record<string, number>`.
- **Suggestion**: For required-key Records (like `Record<KnowledgeCategory, number>`), extract a shared zero-initialized constant (e.g., `EMPTY_CATEGORY_COUNTS`) and use it in all return paths. For accumulator patterns, supply the generic type to `.reduce<Record<...>>()` instead of casting the empty seed.
- **Estimated scope**: 3-4 files, ~6 instances of `{} as Record<...>`.

## Pattern 9: Duplicate `DatabaseError` and other sibling error interfaces across co-located error files

- **Seen in**: slices 3, 26
- **Category**: duplicate-type
- **Combined impact**: low -- structurally identical error interfaces are defined independently in sibling files (`morning-brief.errors.ts` and `proactive.errors.ts` both define `DatabaseError`; `platform-operations.use-case.ts` and `platform-operations.helpers.ts` both define `ResolvedIntegration`). While not dangerous, it creates maintenance burden and confusion about which import to use.
- **What's happening**: When two files in the same domain need the same error or helper type, each defines its own copy rather than sharing one.
- **Suggestion**: For each pair, keep the definition in one file and re-export from the other. Prefer the more foundational file (e.g., the shared errors file or the helpers file) as the canonical location.
- **Estimated scope**: 3-4 pairs of files, each requiring a delete + re-export.

## Pattern 10: `unknown[]` parameter types forcing downstream `as` casts for well-known array element types

- **Seen in**: slices 27, 30
- **Category**: type-cast
- **Combined impact**: medium -- when a function parameter is typed as `unknown[]` but every caller passes `OvernightMessage[]` (or similar known type), every downstream access requires an `as` cast that the compiler cannot verify.
- **What's happening**: `morning-brief.service.ts` (slice 27) declares `highPriorityMessages: unknown[]` and `overnightMessages: unknown[]` in the `createNotification` context parameter, forcing `(m as { id: string }).id` casts at three call sites. `suggestion-feedback.repository.ts` (slice 30) accepts `suggestedAction: unknown` and `modifications: unknown` where contract types (`CompoundSuggestion`) are already imported in the same file.
- **Suggestion**: Replace `unknown` with the actual known type at the parameter boundary. The `unknown` typing provides no safety benefit when every downstream use immediately casts it away -- it just moves the unsafety from the parameter to the access site.
- **Estimated scope**: 2-3 files, ~5 parameters to narrow.
