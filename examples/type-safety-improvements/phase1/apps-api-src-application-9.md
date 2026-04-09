# Audit: apps-api-src-application-9

**Files inspected**: 8
**Findings**: 4

## Summary

These files cover billing rollover utilities, Lemon Squeezy cancel mapping, provider product config, usage history bucketing, and the calendar config domain (errors, repositories, service). The main type-safety issues are: a locally-defined `UsageHistoryTransaction` shape whose `type` field is typed as `string` when a narrower `ActionTransactionType` enum is already exported from the DB layer; a `Record<string, unknown>` return type on `buildRolloverMetadata` that could be a concrete inline type; a `ProviderProduct` interface that duplicates a subset of the already-exported `SubscriptionProductConfig`; and a pair of private interfaces in `CalendarConfigService` (`UpdateCalendarConfigBody` / `CalendarConfigResponse`) that duplicate the DB table shape instead of being derived from it.

## Findings

### Finding 1: `UsageHistoryTransaction.type` typed as `string` instead of `ActionTransactionType`

- **File**: `apps/api/src/application/billing/utils/usage-history.utils.ts:7`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `UsageHistoryTransaction.type` is `string`, but the database enum `actionTransactionTypeEnum` already exports `ActionTransactionType = "subscription_grant" | "usage" | "top_up" | "rollover" | "expiry" | "refund" | "adjustment"`. The filtering logic at line 91 (`t.type === "usage"`) would silently accept any string at the type level and would give no compiler error if the comparison value is misspelled.
- **Suggestion**: Import `ActionTransactionType` from `@/shared/db/tables/enums` and replace `type: string` with `type: ActionTransactionType`.
- **Evidence**:

```typescript
// usage-history.utils.ts:3-8
export interface UsageHistoryTransaction {
  amount: number;
  balanceAfter: number;
  createdAt: Date;
  type: string;   // <-- should be ActionTransactionType
}

// line 91 — comparison to literal would be unguarded:
.filter((t) => t.type === "usage")
```

---

### Finding 2: `buildRolloverMetadata` return type is `Record<string, unknown>` where a concrete type would be safer

- **File**: `apps/api/src/application/billing/utils/billing-rollover.utils.ts:54`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `buildRolloverMetadata` returns `Record<string, unknown>` and initialises the object as that same loose type. The returned object always contains `rollover: number` and `tier: string`, and optionally `idempotencyKey: string`. Callers get no type-level visibility into the shape of this metadata, so downstream code reading these fields must cast or use unsafe narrowing. The type is also written in the return annotation AND in the variable declaration, doubling the looseness.
- **Suggestion**: Replace with a concrete return type:
  ```typescript
  type RolloverMetadata = { rollover: number; tier: string; idempotencyKey?: string };
  ```
  Use this as both the return annotation and the variable type. The `Record<string, unknown>` on the variable would then no longer be needed, because `metadata["idempotencyKey"] = ...` can be replaced with a direct object spread or conditional object literal.
- **Evidence**:

```typescript
// billing-rollover.utils.ts:46-60
export function buildRolloverMetadata({
  tier,
  rollover,
  idempotencyKey,
}: {
  tier: string;
  rollover: number;
  idempotencyKey?: string | undefined;
}): Record<string, unknown> {
  const metadata: Record<string, unknown> = { rollover, tier };
  if (idempotencyKey) {
    metadata["idempotencyKey"] = idempotencyKey;
  }
  return metadata;
}
```

---

### Finding 3: `ProviderProduct` duplicates a subset of `SubscriptionProductConfig` from contracts

- **File**: `apps/api/src/application/billing/utils/provider-product-config.ts:19`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: `ProviderProduct` defines `{ billingPeriod: "monthly" | "annual"; sku: SubscriptionProductSku; tier: "starter" | "pro" | "max" }`. This is identical to three fields of the already-exported `SubscriptionProductConfig` interface from `@slopweaver/contracts`. The `billingPeriod` and `tier` literal unions are also already exported as `BillingPeriod` and `SubscriptionTier` from `@slopweaver/contracts/billing/schemas`. Any change to the canonical values in contracts would not automatically propagate to `ProviderProduct`, creating drift risk.
- **Suggestion**: Replace the hand-rolled interface with a `Pick` of `SubscriptionProductConfig`, and use the canonical contract types for the member literals:
  ```typescript
  import type { SubscriptionProductConfig } from "@slopweaver/contracts";
  export type ProviderProduct = Pick<SubscriptionProductConfig, "billingPeriod" | "sku" | "tier">;
  ```
  This eliminates the duplicate literal unions and keeps `ProviderProduct` in sync automatically.
- **Evidence**:

```typescript
// provider-product-config.ts:19-23
export interface ProviderProduct {
  billingPeriod: "monthly" | "annual";   // duplicates BillingPeriod
  sku: SubscriptionProductSku;
  tier: "starter" | "pro" | "max";       // duplicates SubscriptionTier
}

// contracts/billing/constants.ts:72-79 (already exported)
export interface SubscriptionProductConfig {
  sku: SubscriptionProductSku;
  tier: "starter" | "pro" | "max";
  billingPeriod: "monthly" | "annual";
  ...
}
```

---

### Finding 4: `CalendarConfigResponse` duplicates the Drizzle `UserCalendarConfig` inferred type

- **File**: `apps/api/src/application/calendar/services/calendar-config.service.ts:28`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `CalendarConfigResponse` (lines 28–39) manually restates every field that is already in `UserCalendarConfig` (inferred from `userCalendarConfigTable.$inferSelect`) plus a single additional computed field `connected: boolean`. The `provider` union `"google" | "microsoft" | "apple"` duplicates the `CalendarProvider` type already exported from the table file. If the table schema gains or loses a column the service response will silently diverge.
- **Suggestion**: Derive `CalendarConfigResponse` from the table type instead of duplicating it:

  ```typescript
  import type { UserCalendarConfig, CalendarProvider } from "@/shared/db/tables/user-calendar-config.table";

  type CalendarConfigResponse = Omit<UserCalendarConfig, "accessToken" | "refreshToken" | "expiresAt"> & {
    connected: boolean;
    createdAt: string; // serialised to ISO string
    updatedAt: string;
  };
  ```

  This ensures any future schema changes are automatically reflected in the response type and removes the duplicate `"google" | "microsoft" | "apple"` literal union in favour of `CalendarProvider`.

- **Evidence**:

```typescript
// calendar-config.service.ts:28-39
interface CalendarConfigResponse {
  briefingEnabled: boolean;
  briefingMinutesBefore: number;
  connected: boolean; // only new field
  createdAt: string;
  enabledCalendars: string[];
  id: string;
  primaryCalendarId: string | null;
  provider: "google" | "microsoft" | "apple"; // duplicates CalendarProvider
  updatedAt: string;
  userId: string;
}

// user-calendar-config.table.ts:78
export type UserCalendarConfig = typeof userCalendarConfigTable.$inferSelect;
// (contains briefingEnabled, briefingMinutesBefore, createdAt, enabledCalendars,
//  id, primaryCalendarId, provider, updatedAt, userId — all of the above)
```
