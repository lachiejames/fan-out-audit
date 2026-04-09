# Audit: apps-api-src-shared-6

**Files inspected**: 8
**Findings**: 5

## Summary

This slice contains several table files where jsonb columns lack `$type<>` annotations (`feedback.table.ts`, `suggestion-feedback.table.ts`, `suggestion-presentations.table.ts`). The `integrations.table.ts` is well-typed via `$type<ConnectionSyncSettings>` and `$type<PlatformMetadata>`. The `invoices.table.ts` uses plain `text` for status when an enum would be more precise.

## Findings

### Finding 1: `metadata` column in `feedback.table.ts` lacks `$type<>` annotation

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/feedback.table.ts:35`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: The `metadata` jsonb column is declared without any `$type<>` annotation, meaning Drizzle returns `unknown` for this column. Given that this is a feedback table consolidating multiple feedback types, the metadata likely holds type-specific context (e.g., diff distance for work items, token counts for predictions). Having it typed as `unknown` forces all readers to cast.
- **Suggestion**: Define a `FeedbackMetadata` interface (with optional fields per feedback type) and apply `.$type<FeedbackMetadata>()`.
- **Evidence**:

```typescript
metadata: jsonb("metadata"),
```

### Finding 2: `suggestedAction` and `modifications` in `suggestion-feedback.table.ts` have no `$type<>` annotations

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/suggestion-feedback.table.ts:27,32`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: Both `suggestedAction: jsonb("suggested_action").notNull()` and `modifications: jsonb("modifications")` lack `$type<>` annotations. `suggestedAction` is mandatory (`.notNull()`) but returns `unknown`. These columns store AI suggestion payloads — likely the `WorkItem` or action structure from contracts.
- **Suggestion**: Import the relevant action/suggestion type from `@slopweaver/contracts` and apply `.$type<SuggestedAction>()` and `.$type<SuggestionModifications | null>()` respectively.
- **Evidence**:

```typescript
modifications: jsonb("modifications"),
suggestedAction: jsonb("suggested_action").notNull(),
```

### Finding 3: `primarySuggestion` and `alternativeSuggestions` in `suggestion-presentations.table.ts` have no `$type<>` annotations

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/suggestion-presentations.table.ts:18,23`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `primarySuggestion: jsonb("primary_suggestion").notNull()` and `alternativeSuggestions: jsonb("alternative_suggestions").notNull().default([])` both lack `$type<>` annotations. These are the core suggestion payloads shown to users. The `expandedSuggestionIds` column correctly uses `.$type<string[]>()`, making the inconsistency within the same table apparent.
- **Suggestion**: Import the suggestion type from contracts (or the same type used in `suggestion-feedback.table.ts`) and apply `.$type<Suggestion>()` and `.$type<Suggestion[]>()`.
- **Evidence**:

```typescript
alternativeSuggestions: jsonb("alternative_suggestions").notNull().default([]),
primarySuggestion: jsonb("primary_suggestion").notNull(),
```

### Finding 4: `status` in `invoices.table.ts` typed as plain `text` when a union is known

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/invoices.table.ts:37`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The `status` column has a comment "e.g., 'paid', 'pending', 'refunded'" but uses plain `text`, meaning any string is accepted. The comment implies a closed set of known values from Lemon Squeezy's invoice status enum.
- **Suggestion**: Apply `.$type<"paid" | "pending" | "refunded" | "void">()` (or the full Lemon Squeezy set) to provide compile-time narrowing for reads.
- **Evidence**:

```typescript
status: text("status").notNull(), // e.g., "paid", "pending", "refunded"
```

### Finding 5: `providerMetadata` in `entitlements.table.ts` lacks `$type<>` annotation

- **File**: `/Users/lachiejames/dev/slopweaver/apps/api/src/shared/db/tables/entitlements.table.ts:33`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The `providerMetadata` jsonb column has no `$type<>` annotation. This column stores provider-specific billing metadata from Apple, Google, and Lemon Squeezy. Each provider has a different payload shape, making a union type ideal.
- **Suggestion**: Define a `ProviderMetadata` discriminated union or at minimum `Record<string, unknown>` with a JSDoc comment. If the Lemon Squeezy webhook payload type is available from the `lemon-squeezy` SDK, import and use it.
- **Evidence**:

```typescript
providerMetadata: jsonb("provider_metadata"),
```
