# Cross-Cutting Patterns (Group 3)

**Slices analyzed**: apps-api-src-application-4, apps-api-src-application-31, apps-api-src-application-32, apps-api-src-application-33, apps-api-src-application-34, apps-api-src-application-35, apps-api-src-application-36, apps-api-src-application-37, apps-api-src-application-38, apps-api-src-application-39, apps-api-src-application-40, apps-api-src-application-41

## Pattern 1: `Record<string, unknown>` used as Drizzle `.set()` update accumulator, bypassing column-level type safety

- **Seen in**: slice-31 (suggestions.service.ts byPlatform), slice-33 (test-fixtures-messages.service.ts updateMessage), slice-34 (test-fixtures.service.ts setBillingState, test-fixtures.profile-insert.utils.ts ResolvedProfileFixtures), slice-35 (todo.service.ts updateTodo), slice-36 (training.service.ts style preferences update)
- **Category**: record-weakening
- **Combined impact**: high -- at least 6 independent sites build `Record<string, unknown>` objects and pass them to Drizzle `.set()` calls. A typo in any key name or a wrong value type compiles silently. This is a systemic pattern, not isolated mistakes.
- **What's happening**: When services need to build a dynamic update payload (conditional fields based on input), they declare `const payload: Record<string, unknown> = {}`, populate it with string-keyed assignments, and pass it to `.set(payload)`. Drizzle's `.set()` accepts the table's typed insert/update type, but `Record<string, unknown>` satisfies `any` object constraint and skips all column-name and value-type validation.
- **Suggestion**: Establish a project convention: dynamic update accumulators must be typed as `Partial<typeof table.$inferInsert>` (or a narrower `Pick`/`Omit` variant). This preserves Drizzle's compile-time column validation while still allowing conditional field population. A lint rule or code review checklist item could enforce this.
- **Estimated scope**: ~6 files, ~6-8 instances

## Pattern 2: Local type aliases that duplicate types already exported from `@slopweaver/contracts`

- **Seen in**: slice-4 (TimeRange, Trend/TrendDirection), slice-33 (FixtureWorkItemStatus, FixtureTodoStatus), slice-35 (TodoCategory, TodoPriority, TodoEffort, OriginalTodo), slice-36 (StylePreferences x2, TokenUsageStatsResponse), slice-37 (LearnedRule, LearningProgressCandidate, OverrideStatsResponse), slice-39 (VoiceCatalogProfileEntry, VoiceCatalogPayload, VoiceBillingState inline), slice-41 (SubscriptionStatus inline in 8+ files)
- **Category**: duplicate-type
- **Combined impact**: high -- at least 20+ local type definitions across 8 slices duplicate types that `@slopweaver/contracts` already exports. This is the single most pervasive type-safety issue in the group. When a contract schema evolves, these local copies silently diverge with no compiler error.
- **What's happening**: Developers define local interfaces or type aliases (sometimes with identical field names and literal unions, sometimes with slightly different names like `OriginalTodo` vs `ExtractedTodo` or `StylePreferenceLevels` vs `StylePreferences`) instead of importing from the canonical contracts package. Some files even have comments like "matches contract schema" acknowledging the duplication.
- **Suggestion**: Systematic sweep: for each local type alias whose literal members match a contracts export, replace with an import. Add an ESLint rule or code review convention: "If `@slopweaver/contracts` exports a type with the same shape, import it -- do not redefine." For types that are intentionally subsets (e.g., `Extract<TodoStatus, "pending">`), use the contracts type with a utility type rather than re-declaring the literals.
- **Estimated scope**: ~15-20 files, ~25+ type definitions to replace with imports

## Pattern 3: `Record<string, ...>` maps with known fixed key sets instead of const-keyed or union-keyed types

- **Seen in**: slice-4 (METRIC_THRESHOLDS, buildPlatformBreakdown), slice-31 (SUGGESTION_PREFIX_MAP, byPlatform), slice-32 (CATEGORY_LABELS), slice-33 (buildCategoryStats), slice-35 (todo.service.ts), slice-36 (categoryCounts, byCategory in training), slice-38 (TODO_PRIORITY_MAP), slice-40 (STT_MIME_TO_EXTENSION)
- **Category**: record-weakening
- **Combined impact**: high -- at least 10 `Record<string, ...>` declarations where the key set is a known, finite set of literals. This defeats exhaustiveness checking, allows typo keys to pass silently, and forces downstream `?? fallback` patterns or `as` casts.
- **What's happening**: Constant maps and accumulators are typed with `Record<string, T>` even when the keys are always drawn from a known set (e.g., platform IDs, metric IDs, MIME types, category names, suggestion prefixes). TypeScript's `as const` inference is sometimes present on the value but nullified by the explicit `Record<string, T>` annotation.
- **Suggestion**: For constant lookup tables: remove the `Record<string, T>` annotation and let `as const` infer the narrow type, or use `satisfies Record<string, T>` to check compatibility while preserving the narrow inferred key type. For accumulators built from DB results: type as `Partial<Record<KnownKeyUnion, T>>`. Derive key unions from `keyof typeof MAP` or import from contracts where a canonical union exists (e.g., `PlatformId`, `MetricId`).
- **Estimated scope**: ~10 files, ~12-15 instances

## Pattern 4: Unsafe `as SomeType` casts on JSONB/`unknown` Drizzle columns without runtime validation

- **Seen in**: slice-31 (CompoundSuggestion cast x7, StoredPrediction.predictedActions/contentSignals), slice-33 (as keyof PatternsByType), slice-34 (as any on all 32 fixture arrays, double-cast as any as NewContent[]), slice-35 (as TodoCategory/TodoEffort/TodoPriority from text columns), slice-40 (source string cast to seed union)
- **Category**: type-cast
- **Combined impact**: high -- JSONB columns typed as `unknown` in Drizzle are cast to domain types with bare `as` assertions throughout the codebase. If stored data drifts from the expected schema (e.g., after a contract change or migration), these casts silently produce malformed objects that fail at runtime with no error surface at the point of read.
- **What's happening**: Drizzle JSONB columns are inferred as `unknown`. Services cast them to domain types (`as CompoundSuggestion`, `as PredictionContentSignals`, etc.) at read time without Zod validation. Similarly, `text()` columns that hold enum-like values are cast to their domain union types. The test-fixtures layer takes this to the extreme with `as any` on every fixture record.
- **Suggestion**: Two-pronged fix: (1) For JSONB columns with known schemas, add Zod validation at the repository layer (parse with the contract schema before returning to the service) or use Drizzle's `.$type<T>()` column modifier to declare the expected type at schema level. (2) For text columns that hold enum values, migrate to `pgEnum` columns so Drizzle infers the correct type. For the test-fixtures layer specifically, fixing `ResolvedProfileFixtures` to carry proper Drizzle insert types eliminates all 30+ `as any` casts.
- **Estimated scope**: ~8-10 files, ~40+ cast instances (30+ in test-fixtures alone)

## Pattern 5: Redundant `typeof result === "object"` guard + `as { usage?: unknown }` cast on AI SDK `generateText` result

- **Seen in**: slice-35 (todo-extraction.service.ts x2), slice-36 (training-ai.service.ts)
- **Category**: type-cast
- **Combined impact**: medium -- the exact same defensive pattern is copy-pasted in 3 locations across 2 services. The AI SDK's `GenerateTextResult` already types `usage` as `LanguageModelUsage`, making the guard and cast both redundant and misleading.
- **What's happening**: After calling `generateText()`, the code wraps the result in a `typeof result === "object" && result !== null && "usage" in result` guard, then casts to `{ usage?: unknown }` before extracting usage. This suggests the `extractInputOutputTokensFromUsage` helper accepts `unknown` for its `usage` parameter, forcing all callers to perform this ceremony. The pattern appears to have been copy-pasted from a template.
- **Suggestion**: Fix `extractInputOutputTokensFromUsage` to accept `LanguageModelUsage` (the SDK's type for `usage`) instead of `unknown`. Then all three call sites can simply pass `result.usage` directly, eliminating 3 guards and 3 casts.
- **Estimated scope**: 3 files, 3 instances (but fixing the helper signature is the single root cause)

## Pattern 6: Duplicate error module structures (identical error shapes copy-pasted across domain boundaries)

- **Seen in**: slice-32 (ContextStatusErrors vs SystemStatusErrors -- identical structure, different prefix), slice-39 (VoiceTranscriptionError vs VoiceSynthesisError -- 5 identical provider-error variants)
- **Category**: duplicate-type
- **Combined impact**: medium -- error modules follow a pattern of copy-paste-rename where shared error variants (e.g., `INTERNAL_ERROR`, provider errors like `QUOTA_EXCEEDED`, `RATE_LIMITED`, `UNAVAILABLE`, `UNAUTHORIZED`, `UPSTREAM_FAILURE`) are duplicated verbatim across module boundaries. Adding a field (e.g., `retryAfter` on rate-limited errors) requires updating every copy.
- **What's happening**: The error module pattern (const ErrorCode + base interface + variant interfaces + union type + factory object) is replicated wholesale for each domain module. When two modules share error variants, those variants are copy-pasted rather than composed from a shared base.
- **Suggestion**: Extract shared error variants into composable pieces: (1) A `SharedInternalError` base in `shared/errors/` for the common `INTERNAL_ERROR` pattern. (2) A `ProviderError` union for the five shared voice-provider error variants. Domain error unions can then compose via `type VoiceTranscriptionError = VoiceTranscriptionSpecificError | VoiceProviderError`. Factory functions for shared variants need only be written once.
- **Estimated scope**: ~4 error files, ~10 duplicated variant definitions

## Pattern 7: `string` used for domain fields where contracts export a narrower union type

- **Seen in**: slice-37 (RecordOverrideParams.aiSuggestion/userAction as string instead of TriageAction, LearnedRule.action as string), slice-38 (TriageSessionForStats.status as string, TODO_PRIORITY_MAP keyed by string), slice-40 (voice-vocabulary source as string instead of SeedSource union), slice-41 (WorkItemSource.type as string | null)
- **Category**: missing-strict-typing
- **Combined impact**: medium -- multiple service interfaces and local types use `string` for fields that only ever hold values from a known union. This defeats compile-time validation at call sites and forces downstream casts or duck-type checks.
- **What's happening**: Parameters, interface fields, and map keys are typed as `string` when the actual domain of valid values is a small fixed union (e.g., `TriageAction`, `TriageSessionStatus`, `SeedSource`, content source types). The string values are often compared against typed union values downstream, but TypeScript cannot verify the comparison because one side is `string`.
- **Suggestion**: For each `string` field: identify the canonical union type (usually already in `@slopweaver/contracts` or derivable from a Drizzle enum), and narrow the field type. Where the input genuinely comes from an untyped source (e.g., regex capture, DB text column), validate with Zod at the boundary rather than casting downstream.
- **Estimated scope**: ~6-8 files, ~10-12 field declarations

## Pattern 8: `Record<string, unknown>` return types on functions that build structured objects from typed inputs

- **Seen in**: slice-31 (buildTypedDetails returns Record<string, unknown>), slice-33 (buildCategoryStats returns Record<string, number>), slice-39 (shapeVoiceObservabilityEvent returns Record<string, unknown>)
- **Category**: missing-strict-typing
- **Combined impact**: medium -- utility functions that accept strongly-typed inputs deliberately return `Record<string, unknown>`, erasing all type information at the function boundary. Callers must then cast or narrow the result to use it, often requiring `as SomeType` that the function already had the information to provide.
- **What's happening**: Builder/shaper functions take typed domain objects and produce `Record<string, unknown>` outputs, typically because the consumer (logger, Drizzle `.set()`, contract response) accepts a wide type. But the function boundary erases structure that callers downstream need, forcing casts at every consumption site.
- **Suggestion**: Return the narrowest type the function can provide. If the consumer needs `Record<string, unknown>`, the caller can widen at the call site (TypeScript allows this implicitly for object types). For loggers that accept `Record<string, unknown>`, the function can return `{ msg: string } & Partial<DomainType>` which is assignable to `Record<string, unknown>` but preserves structure for typed callers.
- **Estimated scope**: ~3-4 files, ~4-5 functions
