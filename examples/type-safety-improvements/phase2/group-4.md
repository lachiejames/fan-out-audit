# Cross-Cutting Patterns (Group 4)

**Slices analyzed**: apps-api-src-application-42, apps-api-src-application-5, apps-api-src-application-6, apps-api-src-application-7, apps-api-src-application-8, apps-api-src-application-9, apps-api-src-bootstrap, apps-api-src-cli, apps-api-src-domain-1, apps-api-src-domain-2, apps-api-src-domain-3, apps-api-src-domain-4

## Pattern 1: Local interfaces duplicate types already exported from `@slopweaver/contracts` or Zod schemas

- **Seen in**: apps-api-src-application-5 (Finding 2: `UserResponse`), apps-api-src-application-6 (Findings 1, 3, 6: `CostSummary`/`SavingsBreakdown`, `ResponseTimeByHour`, `ResponseSpeedByRecipient`/`FormalityByRecipient`), apps-api-src-application-7 (Finding 3: `AppleSignedTransaction`), apps-api-src-application-9 (Findings 3, 4: `ProviderProduct`, `CalendarConfigResponse`)
- **Category**: duplicate-type
- **Combined impact**: high -- at least 10 local interfaces across 4 slices duplicate shapes that already exist in the contracts package or adjacent Zod schemas. Each duplicate is a drift risk: when the contract schema changes, the local copy silently lags behind. The `ResponseTimeByHour` case (slice 6, Finding 3) already demonstrates active divergence (local uses `Record<number, number>` while the contract uses `Record<string, number>`), proving this is not theoretical.
- **What's happening**: Services define private or exported interfaces that restate the same fields already captured by Zod schemas in `@slopweaver/contracts` or by Drizzle table `$inferSelect` types. The duplicates typically arise when a service author needs a return type and writes one inline rather than importing the canonical type.
- **Suggestion**: Establish a codebase rule: service return types must be derived from contract schemas (`z.infer<typeof ...>`) or Drizzle table types (`$inferSelect` / `$inferInsert`), never hand-rolled. For the 10+ existing duplicates, replace each with an import or a `Pick`/`Omit` derivation from the canonical source. Prioritize the `ResponseTimeByHour` divergence (already causing casts) and the `CostSummary`/`SavingsBreakdown` pair (high-traffic billing code).
- **Estimated scope**: ~10-12 interfaces across 7+ files

## Pattern 2: `Record<string, unknown>` used for metadata/properties with known keys

- **Seen in**: apps-api-src-application-7 (Finding 5: `billing-actions.service.ts` metadata), apps-api-src-application-9 (Finding 2: `buildRolloverMetadata`), apps-api-src-domain-2 (Findings 3, 4: `ActionDetectionContext.metadata`, `IPostHogPort` properties), apps-api-src-domain-4 (Findings 2, 4: `PlatformSearchError.details`, `NotionQueryDatabaseCommand.filter`), apps-api-src-application-8 (Finding 3: `ServiceAccountPayload.credentials`)
- **Category**: record-weakening
- **Combined impact**: medium -- at least 8 distinct `Record<string, unknown>` usages across 5 slices where the actual keys are known or partially known. The pattern suppresses compile-time checking for known fields, forcing consumers to perform unsafe narrowing.
- **What's happening**: When a value has a mix of known and extensible fields, the code defaults to `Record<string, unknown>` instead of defining the known fields explicitly. This is particularly problematic in `billing-actions.service.ts` where `metadata` always contains `units` and `idempotencyKey` but the type does not reflect this, and in `buildRolloverMetadata` where the return shape is fully known but typed as open.
- **Suggestion**: For each case, define a concrete interface with the known optional fields plus an index signature for extensibility: `{ units?: number; idempotencyKey?: string; [key: string]: unknown }`. For domain ports where the looseness is architecturally intentional (PostHog, Notion filter), add JSDoc documenting expected keys rather than changing the type. Prioritize the billing metadata path (high-traffic, known fields).
- **Estimated scope**: ~8-10 usages across 7+ files

## Pattern 3: `as SomeType` casts on `response.json()` / `sanitizeUtf8Json()` / Zod-parsed output without runtime validation

- **Seen in**: apps-api-src-application-42 (Findings 3, 4, 5: `as ActionPreview` after sanitize/spread), apps-api-src-application-7 (Findings 1, 2: `as T` after JWS verify, `as AppStoreTransactionResponse` after fetch), apps-api-src-application-8 (Finding 5: `as Checkout`/`as Subscription` after fetch), apps-api-src-cli (Finding 1: `as Array<...>` on Supabase listUsers)
- **Category**: type-cast
- **Combined impact**: high -- at least 8 instances across 4 slices where external data (HTTP responses, JWT payloads, SDK results) is cast to a concrete type without Zod parsing or runtime validation. The JWS `as T` cast (slice 7, Finding 1) is the most dangerous: it allows a malformed JWT payload to flow through as a trusted typed value. The `response.json() as Checkout` pattern (slice 8) is also high-risk since external API shape changes would cause silent runtime failures.
- **What's happening**: After calling `response.json()`, `sanitizeUtf8Json()`, or `jwtVerify()`, the code casts the `unknown`/`any` result directly to the expected type. This bypasses TypeScript's purpose at the boundary where runtime validation matters most.
- **Suggestion**: Adopt a boundary validation pattern: every external data ingestion point should parse through a Zod schema before the value enters typed code. For `sanitizeUtf8Json`, add a generic overload that preserves the input type. For `response.json()` calls, define lightweight Zod schemas matching the expected response shape and `.parse()` before use. The existing `parseJson` utility in the notification service is a model to follow.
- **Estimated scope**: ~8-10 cast sites across 5+ files

## Pattern 4: Functions accept `string` where a narrow enum/union type exists, then cast internally

- **Seen in**: apps-api-src-application-42 (Findings 7, 8: `type: string` cast to `IntegrationActionType`/`WorkItemType`), apps-api-src-application-9 (Finding 1: `UsageHistoryTransaction.type` as `string` vs `ActionTransactionType`), apps-api-src-domain-1 (Finding 1: `relationship: string | null` vs contracts union), apps-api-src-domain-4 (Finding 3: `canUndo(actionType: string)`)
- **Category**: missing-strict-typing
- **Combined impact**: medium -- at least 6 functions across 4 slices accept `string` for parameters that are always one of a known enum or literal union, then immediately cast to the narrow type internally. This defeats exhaustiveness checking and lets callers pass invalid strings without compile-time errors.
- **What's happening**: When functions are written, the author uses `string` as a "safe" wide type for parameters whose actual domain is a narrow union (e.g., `WorkItemType`, `ActionTransactionType`, `RelationshipType`). The cast inside the function then narrows it, but the damage is done: callers have no compile-time constraint.
- **Suggestion**: Change parameter types to the narrow enum/union. Since call-sites already have values of the narrow type (confirmed in the audit findings), this is a signature-only change with no runtime impact. Grep for `as IntegrationActionType`, `as WorkItemType`, and similar casts to find all instances systematically.
- **Estimated scope**: ~6-8 function signatures across 5+ files

## Pattern 5: `Set.has()` type workarounds for narrow const sets vs wider enum types

- **Seen in**: apps-api-src-application-42 (Findings 2, 9: `dedupeOperationalTypes.has(... as ...)` and `syncableTypes.has(... as typeof syncableTypes extends Set<infer T> ? T : never)`)
- **Category**: type-cast
- **Combined impact**: medium -- concentrated in 2 instances in the work-items domain, but the pattern likely exists elsewhere. The `Set<"a" | "b"> .has(value: WiderUnion)` problem is a known TypeScript ergonomic issue that the codebase handles with increasingly complex workarounds (conditional type inference in Finding 9).
- **What's happening**: A `Set` is declared with `as const` narrow literal members, but `.has()` is called with a value of a wider type (e.g., `WorkItemType`). TypeScript rejects this because `Set<"a" | "b">.has()` only accepts `"a" | "b"`. The workarounds range from simple casts to a complex `typeof syncableTypes extends Set<infer T> ? T : never` conditional type.
- **Suggestion**: Adopt a single codebase-wide pattern: declare sets with the wider type as element type (`new Set<WorkItemType>([...])`) so `.has()` accepts any `WorkItemType`. This is safe because `.has()` returns `false` for non-members. Alternatively, create a utility `setIncludes<T>(set: ReadonlySet<T>, value: T extends string ? string : T): value is T` that handles the narrowing cleanly.
- **Estimated scope**: ~3-5 instances (2 confirmed, likely more in un-audited files)

## Pattern 6: SDK type duplication -- hand-rolled interfaces mirroring external SDK types

- **Seen in**: apps-api-src-application-5 (Finding 1: `CookieOptions` vs Express), apps-api-src-application-8 (Findings 1, 2: `GooglePlaySubscriptionResponse`/`GooglePlayProductPurchaseResponse` vs `googleapis`), apps-api-src-cli (Finding 2: `AuthUserRecord` vs Supabase `User`)
- **Category**: sdk-type-duplication
- **Combined impact**: high -- at least 4 hand-rolled interfaces across 3 slices duplicate types already exported by their respective SDKs (Express, googleapis, Supabase). The Google Play types are the highest-risk: the custom `GooglePlaySubscriptionResponse` includes fields (`lineItems[].startTime`, `lineItems[].orderId`) that may not exist on the actual SDK type `Schema$SubscriptionPurchaseV2`, meaning the code may be accessing fields that do not exist at runtime.
- **What's happening**: When integrating with external SDKs, authors create local interfaces with only the fields they need rather than importing the SDK type and using `Pick`. This creates a lossy subset that misses SDK updates, and the accompanying `as CustomType` casts on SDK return values hide the real types.
- **Suggestion**: Replace custom interfaces with `Pick<SDKType, "needed" | "fields">` or use the SDK type directly. For domain ports that intentionally avoid SDK imports (architectural boundary), keep the local type but add a JSDoc `@see` reference to the SDK type it mirrors. Prioritize the Google Play types (active field mismatch risk).
- **Estimated scope**: ~4-5 interfaces across 4+ files

## Pattern 7: Positional parameters on exported functions (violating named-params rule)

- **Seen in**: apps-api-src-application-6 (Finding 2: `canProceedWithAICall(userId)`), apps-api-src-domain-1 (Finding 2: `AttachmentErrors` factory functions), apps-api-src-domain-4 (Finding 3: `canUndo(actionType)`)
- **Category**: missing-strict-typing
- **Combined impact**: low -- at least 3 sites across 3 slices use positional params where the codebase mandates named object params. The `AttachmentErrors` factory is the worst offender with multiple 2-3 arg positional functions.
- **What's happening**: Some exported functions and abstract method signatures use positional parameters instead of the codebase's required `{ param }: { param: Type }` pattern. This makes parameter order errors invisible to callers and is inconsistent with the majority of the codebase.
- **Suggestion**: Refactor to named params. These are mechanical changes: update the signature and all call-sites. Can be done as a batch PR.
- **Estimated scope**: ~5-8 function signatures across 3+ files

## Pattern 8: Domain port return types use bare `unknown` where a minimal structural contract would help

- **Seen in**: apps-api-src-domain-2 (Findings 1, 2: `AgentStreamResult` tool args/results as `unknown`, `DomainTool.execute` args as `unknown`), apps-api-src-domain-3 (Findings 1, 2: `ThreadData = unknown`, `FetchThreadResult = unknown`, `SendReplyResult` with `Record<string, unknown>`)
- **Category**: any-usage
- **Combined impact**: medium -- at least 5 type aliases or fields across 2 slices use bare `unknown` for values that always have at minimum an object shape. Every consumer must perform unsafe narrowing with no type-level guidance. This is architecturally intentional (domain ports avoid SDK imports) but the current approach is more restrictive than necessary.
- **What's happening**: Domain ports define return types as `unknown` to avoid leaking SDK types across the hexagonal architecture boundary. However, the actual runtime values are always JSON-serializable objects, never primitives. Using `unknown` instead of `Record<string, unknown>` or a minimal structural type forces every consumer to narrow from scratch.
- **Suggestion**: Replace bare `unknown` with the minimum structural contract that holds across all platform implementations. For `ThreadData`, a discriminated union `{ platform: string } & Record<string, unknown>` is a safe minimum. For `AgentStreamResult` tool args, `Record<string, unknown>` is always correct since AI SDK tool args are JSON objects. This gives consumers something to destructure without importing SDK types.
- **Estimated scope**: ~5-6 type aliases across 4+ files
