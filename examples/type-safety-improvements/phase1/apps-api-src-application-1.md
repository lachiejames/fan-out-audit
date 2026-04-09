# Audit: apps-api-src-application-1

**Files inspected**: 8
**Findings**: 5

## Summary

These files cover account-deletion (errors + service), activity (errors, stream service, CRUD service), and three agent use-cases (fetch-content, manage-calendar, manage-document-store). The code is generally well-typed with consistent Result/neverthrow patterns and discriminated-union errors. The main issues are: (1) a duplicated, narrower issue shape in `FetchLinearIssueResult` that diverges from the port's `LinearIssueResult`, (2) three `as const` casts that are necessary only because the `sourceType` field is widened unnecessarily at the call-site, (3) non-null assertions on `Array.find()` results that could crash at runtime, (4) an overly wide `platform: string` param where a constrained union exists, and (5) `Record<string, unknown>` on `FetchContentError.details` where a stricter union would eliminate the `Record`.

## Findings

### Finding 1: `FetchLinearIssueResult` duplicates a narrower subset of `LinearIssueResult`

- **File**: `apps/api/src/application/agents/use-cases/fetch-content.use-case.ts:91`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `FetchLinearIssueResult` in the use-case defines its own `issue` shape (5 fields, all required or null) that substantially overlaps with `LinearIssueResult` in the port (`content-fetch.port.ts:37`). The port type is richer (identifier, url, dueDate, estimate, priorityLabel, creator, labels, comments, pagination) and is what adapters actually return. The use-case result is then constructed with `{ ...issueData, integrationId }` where `issueData` is `LinearIssueResult`, so the spread already satisfies the narrower type — but callers only see the truncated interface, losing access to `comments`, `labels`, `identifier`, etc. This creates silent data loss at the type boundary.
- **Suggestion**: Replace the inline `issue` shape in `FetchLinearIssueResult` with `LinearIssueResult` imported from the port, then add `integrationId`:

  ```typescript
  import type { LinearIssueResult } from "@/domain/ports/integrations/content-fetch.port";

  export interface FetchLinearIssueResult extends LinearIssueResult {
    integrationId: string;
  }
  ```

- **Evidence**:

```typescript
// fetch-content.use-case.ts:91-102
export interface FetchLinearIssueResult {
  integrationId: string;
  issue: {
    id: string;
    title: string;
    description: string | null;
    state: { name: string } | null;
    priority: number | null;
    assignee: { name: string } | null;
  };
}

// content-fetch.port.ts:37-61 — the richer canonical shape
export interface LinearIssueResult {
  issue: {
    id: string;
    identifier?: string;
    title: string;
    description: string | null;
    url?: string;
    // ...dueDate, estimate, state with color/type, priorityLabel, creator...
  };
  labels?: { name: string; color?: string }[];
  comments: { id: string; body: string; createdAt: string; ... }[];
  pagination?: { hasMore: boolean };
}
```

---

### Finding 2: `as const` casts on `sourceType` literals in `ActivityStreamService`

- **File**: `apps/api/src/application/activity/services/activity-stream.service.ts:166`
- **Category**: type-cast
- **Impact**: low
- **Description**: The `sourceType` field is set via conditional spreads like `{ sourceType: "integration_message" as const }` in three separate methods (lines 166, 227, 285). The `as const` is required because without it TypeScript widens the string literal to `string`, which is incompatible with the `ActivityStreamEvent["sourceType"]` union. The root cause is that the spread expression `{}` loses the type, forcing the cast. This is a recurring pattern across `emitWorkItemGenerated`, `emitTodoExtracted`, and `emitKnowledgeExtracted`.
- **Suggestion**: Extract the conditional sourceType assignment into a typed intermediate variable before the spread, allowing TypeScript to infer the literal without a cast:
  ```typescript
  const sourceTypeField = sourceId !== undefined
    ? ({ sourceId, sourceType: "integration_message" } satisfies Pick<ActivityStreamEvent, "sourceId" | "sourceType">)
    : {};
  return this.emitActivityEvent({ ..., ...sourceTypeField, ... });
  ```
  Alternatively, add explicit `sourceType` and `sourceId` optional params to `emitWorkItemGenerated` and always pass them (passing `undefined` is valid for optional fields), letting the base `emitActivityEvent` handle the null coalescing — eliminating all three casts.
- **Evidence**:

```typescript
// activity-stream.service.ts:162-168
return this.emitActivityEvent({
  ...
  ...(messageId !== undefined ? { actionLabel: "View Message" } : {}),
  ...(messageId !== undefined ? { actionUrl: `/inbox?messageId=${messageId}` } : {}),
  ...(messageId !== undefined ? { sourceId: messageId } : {}),
  ...(messageId !== undefined ? { sourceType: "integration_message" as const } : {}),  // cast required
});
// Same pattern at lines 227 and 285
```

---

### Finding 3: Non-null assertions on `Array.find()` in `getTodayEvents`

- **File**: `apps/api/src/application/agents/use-cases/manage-calendar.use-case.ts:439`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: Three consecutive non-null assertions (`!`) are used on `.find()` results from `Intl.DateTimeFormat.formatToParts()`. While `formatToParts` for `"en-US"` locale with `year/month/day` options always includes those parts, `!` silently bypasses the `undefined` case. If the locale or options ever change (e.g., a future refactor), these become runtime exceptions (`Cannot read properties of undefined`). The TypeScript return type of `find()` is `T | undefined`, and asserting it away without a guard is unsafe.
- **Suggestion**: Replace the non-null assertions with nullish coalescing or explicit guards:
  ```typescript
  const year =
    parts.find((p) => p.type === "year")?.value ??
    (() => {
      throw new Error("Missing year part");
    })();
  ```
  Or more practically, use a helper that validates all three parts are present before proceeding:
  ```typescript
  const year = parts.find((p) => p.type === "year")?.value;
  const month = parts.find((p) => p.type === "month")?.value;
  const day = parts.find((p) => p.type === "day")?.value;
  if (!year || !month || !day) {
    // fall back to UTC path
    endOfDay = new Date(now);
    endOfDay.setUTCHours(23, 59, 59, 999);
  } else {
    // use year/month/day
  }
  ```
  This mirrors the existing `catch` fallback already in place.
- **Evidence**:

```typescript
// manage-calendar.use-case.ts:439-441
const year = parts.find((p) => p.type === "year")!.value; // non-null assertion
const month = parts.find((p) => p.type === "month")!.value; // non-null assertion
const day = parts.find((p) => p.type === "day")!.value; // non-null assertion
```

---

### Finding 4: `platform: string` in `resolveIntegrationId` should use a constrained union

- **File**: `apps/api/src/application/agents/use-cases/fetch-content.use-case.ts:328`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The private `resolveIntegrationId` method accepts `platform: string`, but the three callers always pass one of three specific literals: `"slack"`, `"linear"`, `"google-gmail"`. These are valid `PlatformId` values from `@slopweaver/contracts`. Using `string` means a typo like `"google_gmail"` compiles silently and produces a confusing runtime error ("No slack integration connected" style). The parameter should be constrained to `PlatformId` or at minimum a local union of the three supported values.
- **Suggestion**:

  ```typescript
  import type { PlatformId } from "@slopweaver/contracts";

  private async resolveIntegrationId({
    userId,
    platform,
    providedIntegrationId,
  }: {
    userId: string;
    platform: PlatformId;
    providedIntegrationId?: string | undefined;
  }): Promise<Result<string, FetchContentError>>
  ```

- **Evidence**:

```typescript
// fetch-content.use-case.ts:323-331
private async resolveIntegrationId({
  userId,
  platform,
  providedIntegrationId,
}: {
  userId: string;
  platform: string;   // <-- too wide
  providedIntegrationId?: string | undefined;
}): Promise<Result<string, FetchContentError>>
```

---

### Finding 5: `FetchContentError.details` typed as `Record<string, unknown>` where a discriminated shape would be safer

- **File**: `apps/api/src/application/agents/use-cases/fetch-content.use-case.ts:65`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `FetchContentError` carries an optional `details?: Record<string, unknown>` field. In practice there is only one usage site (line 362-368) where `details` is populated, and it always has the shape `{ availableIntegrations: { connectionName: string | null; integrationId: string }[] }`. Because the field is `Record<string, unknown>`, callers that need to display or forward `availableIntegrations` must cast or use `unknown` narrowing. A discriminated or inline type would make the `MULTIPLE_INTEGRATIONS` case self-documenting and safe.
- **Suggestion**: Narrow the type per error code. The simplest approach is to add an overloaded constructor or use a discriminated error class, but the minimal fix is:
  ```typescript
  export class FetchContentError extends Error {
    constructor(
      public readonly type: FetchContentErrorType,
      message: string,
      public readonly details?: type extends "MULTIPLE_INTEGRATIONS"
        ? { availableIntegrations: { connectionName: string | null; integrationId: string }[] }
        : Record<string, unknown>,
    ) { ... }
  }
  ```
  Or keep the class simple and add a typed static factory:
  ```typescript
  static multipleIntegrations(
    message: string,
    availableIntegrations: { connectionName: string | null; integrationId: string }[],
  ): FetchContentError {
    return new FetchContentError("MULTIPLE_INTEGRATIONS", message, { availableIntegrations });
  }
  ```
- **Evidence**:

```typescript
// fetch-content.use-case.ts:61-70
export class FetchContentError extends Error {
  constructor(
    public readonly type: FetchContentErrorType,
    message: string,
    public readonly details?: Record<string, unknown>,  // <-- too wide
  ) { ... }
}

// fetch-content.use-case.ts:361-368 — only usage with populated details
return err(
  new FetchContentError(
    "MULTIPLE_INTEGRATIONS",
    `Multiple ${platform} accounts connected. Please specify integrationId.`,
    {
      availableIntegrations: integrations.map((i) => ({
        connectionName: i.connectionName,
        integrationId: i.id,
      })),
    },
  ),
);
```
