# Audit: packages-contracts-src-contracts-2

**Files inspected**: 8
**Findings**: 7

## Summary

The billing module is large and largely well-typed. The main issues are: one `Record<string, string>` that should have constrained keys, a type cast inside `getMinimumTierForFeature`, loose `string` for `tier` in `paywallErrorDetailsSchema`, a `TRIAL_CONFIG` using a type cast on its `tier` field instead of `satisfies`, and duplicate billing action definitions between `paywall-errors.ts` and `billing/constants.ts`.

## Findings

### Finding 1: `actionTransactionSchema.metadata` is `z.record(z.string(), z.unknown()).nullable()` — opaque payload

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/billing/schemas.ts:159`
- **Category**: any-usage
- **Impact**: medium
- **Description**: The `metadata` field on `ActionTransaction` is `Record<string, unknown> | null`. This field contains action-specific context (e.g., pack IDs, expiry reasons) that is never validated or typed. Frontend/backend both treat this as an opaque blob.
- **Suggestion**: Define a discriminated union or at minimum named sub-schemas for the known transaction types (`subscription_grant`, `usage`, `top_up`, etc.) so callers can narrow the metadata type based on `type`. This is a medium-term improvement.
- **Evidence**: `metadata: z.record(z.string(), z.unknown()).nullable(),` at line 159 in `schemas.ts`.

### Finding 2: `paywallErrorDetailsSchema.tier` is `z.string().optional()` — should be `subscriptionTierSchema`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/billing/paywall-errors.ts:72`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: The `tier` field in `paywallErrorDetailsSchema` is `z.string().optional()` rather than `subscriptionTierSchema.optional()`. The frontend uses this value to route upgrade UX; an invalid string would silently fall through.
- **Suggestion**: Import `subscriptionTierSchema` from `./schemas.ts` and use it: `tier: subscriptionTierSchema.optional()`.
- **Evidence**: `tier: z.string().optional(),` at line 72 of `paywall-errors.ts`.

### Finding 3: `billingActionSchema` in `paywall-errors.ts` duplicates `ACTION_COSTS` keys from `constants.ts`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/billing/paywall-errors.ts:27-46`
- **Category**: duplicate-type
- **Impact**: high
- **Description**: `billingActionSchema` is a manually maintained `z.enum([...])` with 18 values. These same values are the keys of `ACTION_COSTS` in `constants.ts` (exported as `ActionType`). If a new action type is added to `ACTION_COSTS`, `billingActionSchema` must be updated separately — and a mismatch would be silent.
- **Suggestion**: Derive `billingActionSchema` from `ACTION_COSTS` keys:
  ```typescript
  import { ACTION_COSTS } from "./constants.ts";
  export const billingActionSchema = z.enum(
    Object.keys(ACTION_COSTS) as [keyof typeof ACTION_COSTS, ...Array<keyof typeof ACTION_COSTS>],
  );
  ```
  This makes the two definitions single-source.
- **Evidence**: `billingActionSchema` enum at lines 27-46 vs `ACTION_COSTS` object keys in `constants.ts` lines 457-484.

### Finding 4: `getMinimumTierForFeature` uses `as never` cast

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/billing/constants.ts:429-438`
- **Category**: type-cast
- **Impact**: low
- **Description**: The function calls `TIER_FEATURES.free.includes(feature as never)` and similar. The `as never` cast is a workaround for TypeScript's inability to infer the correct overload for `readonly string[].includes`.
- **Suggestion**: Use `(TIER_FEATURES.free as readonly string[]).includes(feature)` (the same pattern used in `isActiveSubscription` elsewhere in this file). This avoids the `never` cast and is more readable.
- **Evidence**: `TIER_FEATURES.free.includes(feature as never)` at line 429.

### Finding 5: `TRIAL_CONFIG.tier` uses inline cast `as SubscriptionTier` rather than `satisfies`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/billing/constants.ts:508`
- **Category**: type-cast
- **Impact**: low
- **Description**: `tier: "pro" as SubscriptionTier` is a type cast that widens the literal `"pro"` to `SubscriptionTier`. If the value were changed to an invalid string, TypeScript would not catch it until runtime.
- **Suggestion**: Use `satisfies SubscriptionTier` or `tier: "pro" as const satisfies SubscriptionTier` to preserve the literal while still checking it's a valid tier.
- **Evidence**: `tier: "pro" as SubscriptionTier,` at line 508 in `constants.ts`.

### Finding 6: `SubscriptionProductConfig.tier` is typed as `"starter" | "pro" | "max"` inline rather than `SubscriptionTier`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/billing/constants.ts:73-74`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: `SubscriptionProductConfig.billingPeriod` is typed as `"monthly" | "annual"` and `tier` as `"starter" | "pro" | "max"` inline, duplicating the `SubscriptionTier` and `BillingPeriod` types already defined in `schemas.ts` and imported at the top of this file.
- **Suggestion**: Use the imported types directly: `tier: SubscriptionTier; billingPeriod: BillingPeriod;`. This ensures any future changes to the enums are reflected automatically.
- **Evidence**: `tier: "starter" | "pro" | "max";` at line 73; `billingPeriod: "monthly" | "annual";` at line 74.

### Finding 7: `invoiceSchema.status` is `z.string()` — should be a narrow enum

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/billing/list-invoices.ts:27`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `status: z.string()` on `invoiceSchema` accepts any string. Lemon Squeezy invoices have known statuses (`paid`, `pending`, `void`, `refunded`). Keeping this as a string means the frontend cannot reliably switch on the status value.
- **Suggestion**: Define a `z.enum(["paid", "pending", "void", "refunded"])` or at minimum add a comment enumerating the known values.
- **Evidence**: `status: z.string(),` at line 27 of `list-invoices.ts`.
