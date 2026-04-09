# Audit: apps-api-src-application-3

**Files inspected**: 8
**Findings**: 4

## Summary

These files cover AI agent capability use-cases (platform search/actions, workspace search, semantic embeddings) and analytics services (overview and timeseries). The agent use-cases are generally well-typed using neverthrow Results and port abstractions. The main findings are a duplicate local interface, untyped `platform: string` parameters where a narrower type is available, `any` usage in the analytics service helper, and an unsafe `Number()` cast on a Drizzle `sql<number>` column that is acknowledged but not typed correctly at source.

## Findings

### Finding 1: Duplicate `ResolvedIntegration` interface

- **File**: `apps/api/src/application/agents/use-cases/platform-operations.use-case.ts:48` and `apps/api/src/application/agents/use-cases/platform-operations.helpers.ts:5`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: The interface `ResolvedIntegration { integrationId: string }` is defined identically in both files. The helper file exports a function that returns `Result<ResolvedIntegration, ...>` using its own local copy, while the use-case file defines another copy as a private `interface`. They are structurally identical and serve the same purpose. Since `platform-operations.use-case.ts` imports the helper function, it could re-use the helper's type rather than silently defining a duplicate.
- **Suggestion**: Export `ResolvedIntegration` from `platform-operations.helpers.ts` and import it in `platform-operations.use-case.ts` instead of redefining it. The private `interface ResolvedIntegration` block at line 48 of the use-case file can then be removed.
- **Evidence**:

```typescript
// platform-operations.helpers.ts:5-7
interface ResolvedIntegration {
  integrationId: string;
}

// platform-operations.use-case.ts:48-50
interface ResolvedIntegration {
  integrationId: string;
}
```

---

### Finding 2: `platform: string` should use `PlatformId` from contracts

- **File**: `apps/api/src/application/agents/use-cases/platform-operations.use-case.ts:744`, `platform-operations.helpers.ts:22`, `platform-actions.use-case.ts:148`, `semantic-search.use-case.ts:42`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: Several internal parameters accept `platform: string` with no constraint, even though `@slopweaver/contracts` already exports `PlatformId` (a union of all valid platform string literals) and `PLATFORM_IDS`. Using the wider `string` type means a typo like `"githb"` or a completely unknown platform can silently pass through without a compile-time error. The `resolveIntegration` helper in `platform-operations.helpers.ts` and the private helper in `platform-actions.use-case.ts` both forward this `platform` value directly to `IIntegrationsLookupPort.findByPlatformForUser({ platform })`. If that port is typed with `PlatformId`, the narrower type would be enforced at the call-site.
- **Suggestion**: Change the `platform: string` parameters in `resolveIntegration` (helpers file), `PlatformActionsUseCase.resolveIntegrationId`, and `GenerateContextualEmbeddingParams` to `platform: PlatformId` (imported from `@slopweaver/contracts`). For `GenerateContextualEmbeddingParams` in `semantic-search.use-case.ts`, keeping it as `string` may be intentional if the embedding platform set differs from the integration platform set — document the reason or narrow the type.
- **Evidence**:

```typescript
// platform-operations.helpers.ts:22
platform: string;

// platform-actions.use-case.ts:148
platform: string;

// semantic-search.use-case.ts:42
platform: string;
```

---

### Finding 3: `table: any` and `conditions: any[]` in `countRows` helper

- **File**: `apps/api/src/application/analytics/services/analytics.service.ts:739-742`
- **Category**: any-usage
- **Impact**: high
- **Description**: The private `countRows` helper method accepts `table: any` and `conditions: any[]`. This completely erases type safety for every call-site — there is no compile-time check that the table has a `userId` column, that conditions are valid Drizzle SQL expressions, or that `eq(table.userId, userId)` will not fail at runtime on an incompatible table. The helper is called 15+ times throughout the service and the `any` typing propagates to all those call-sites.
- **Suggestion**: Use Drizzle's type utilities to constrain the `table` parameter to tables that have a `userId` column. A practical approach is to type it as `{ userId: Column }` (using Drizzle's `Column` type from `drizzle-orm`) and type `conditions` as `SQL[]` or `(SQL | undefined)[]` from `drizzle-orm`. A minimal fix that avoids complex generics:

```typescript
import type { SQL } from "drizzle-orm";
import type { PgTable } from "drizzle-orm/pg-core";

private countRows({
  table,
  userId,
  conditions,
}: {
  table: PgTable & { userId: unknown };
  userId: string;
  conditions: SQL[];
}): ResultAsync<number, DatabaseError>
```

- **Evidence**:

```typescript
// analytics.service.ts:735-742
  private countRows({
    table,
    userId,
    conditions,
  }: {
    table: any;
    userId: string;
    conditions: any[];
  }): ResultAsync<number, DatabaseError> {
```

---

### Finding 4: Unsafe `Number()` cast on Drizzle `sql<number>` columns

- **File**: `apps/api/src/application/analytics/services/analytics.service.ts:722-724`, `analytics.service.ts:766-768`, and `analytics-timeseries.service.ts:574`
- **Category**: type-cast
- **Impact**: low
- **Description**: Drizzle's `sql<number>` type annotation is a lie — Postgres drivers return numeric aggregates as strings at runtime. This is acknowledged via `// eslint-disable-next-line @typescript-eslint/no-unnecessary-type-conversion` comments, which suppress the lint rule rather than fixing the source type. The `Number()` conversions are correct at runtime, but the type `sql<number>` creates a false sense of safety: any code that reads `rows[0]?.count` without the `Number()` conversion will silently receive a string. The `computeResponseTimeSeries` in the timeseries service has the same pattern but the comment correctly acknowledges it.
- **Suggestion**: Change the Drizzle annotation from `sql<number>` to `sql<string>` for `COUNT(DISTINCT ...)` expressions that return strings at runtime. This makes the `Number()` conversion an explicit and required step rather than a suppressed lint violation. Alternatively, create a typed helper `countDistinct(expr)` that returns `sql<string>` and always wraps it with `Number()`. The existing `count()` Drizzle helper already returns a number correctly — only the raw `sql<number>\`COUNT(DISTINCT ...)\`` expressions are affected.
- **Evidence**:

```typescript
// analytics.service.ts:718-724 (countDistinctEditedWorkItems)
      .select({ count: sql<number>`COUNT(DISTINCT ${workItemRevisionsTable.workItemId})` })
      ...
    }).map((rows) => {
      // eslint-disable-next-line @typescript-eslint/no-unnecessary-type-conversion -- Drizzle sql<number> returns string at runtime
      return Number(rows[0]?.count ?? 0);
    });

// analytics.service.ts:762-768 (countTriageSessions)
      .select({ count: sql<number>`COUNT(DISTINCT ${triageOverridesTable.sessionId})` })
      ...
    }).map((rows) => {
      // eslint-disable-next-line @typescript-eslint/no-unnecessary-type-conversion -- Drizzle sql<number> returns string at runtime
      return Number(rows[0]?.count ?? 0);
    });
```
