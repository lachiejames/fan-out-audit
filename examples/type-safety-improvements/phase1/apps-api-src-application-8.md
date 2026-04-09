# Audit: apps-api-src-application-8

**Files inspected**: 8
**Findings**: 5

## Summary

These files implement the billing subsystem: receipt verification for Apple/Google purchases, Lemon Squeezy checkout/subscription management, trial policy enforcement, and shared billing utilities. The main type-safety issues are SDK type duplication in the Google Play service, several unavoidable-looking but worth-noting type casts (Google Play API response coercion and Lemon Squeezy SDK response coercion), a `Record<string, unknown>` for service account credentials that could be tightened, and one unsafe `as` cast for `autoRefillPackId` in billing.service.ts that should use a type guard instead.

## Findings

### Finding 1: Custom `GooglePlaySubscriptionResponse` duplicates `androidpublisher_v3.Schema$SubscriptionPurchaseV2`

- **File**: `apps/api/src/application/billing/services/google-play.service.ts:18`
- **Category**: sdk-type-duplication
- **Impact**: high
- **Description**: The file defines a hand-rolled `GooglePlaySubscriptionResponse` interface and casts `response.data as GooglePlaySubscriptionResponse` at line 112. The `googleapis` package already exports `androidpublisher_v3.Schema$SubscriptionPurchaseV2` which covers all the same fields (`externalAccountIdentifiers`, `lineItems`, `linkedPurchaseToken`, `subscriptionState`, `latestOrderId`). The custom type is a lossy subset: its `lineItems` array only includes `expiryTime`, `productId`, `startTime`, `offerId`, `orderId`, but the SDK's `Schema$SubscriptionPurchaseLineItem` exposes additional fields that callers may need in the future. The local type also accepts `string` for `subscriptionState` (correct — matches SDK), but it doesn't benefit from SDK documentation or future SDK updates.
- **Suggestion**: Replace `GooglePlaySubscriptionResponse` with `androidpublisher_v3.Schema$SubscriptionPurchaseV2` imported from `googleapis`. The `as` cast on line 112 would then become unnecessary — `response.data` is already typed as `Schema$SubscriptionPurchaseV2` by the SDK. Note: the SDK's `Schema$SubscriptionPurchaseLineItem` does **not** include `startTime` or `orderId` fields (the v3 API moved to `latestSuccessfulOrderId`); callers in `billing-receipt.service.ts` that read `lineItem.startTime` and `lineItem.orderId` would need to be reconciled with the actual API shape.
- **Evidence**:

```typescript
// google-play.service.ts:18-33
interface GooglePlaySubscriptionResponse {
  externalAccountIdentifiers?: {
    obfuscatedExternalAccountId?: string;
    obfuscatedExternalProfileId?: string;
  };
  lineItems?: Array<{
    expiryTime?: string;
    productId?: string;
    startTime?: string;
    offerId?: string;
    orderId?: string;
  }>;
  linkedPurchaseToken?: string;
  subscriptionState?: string;
  latestOrderId?: string;
}

// line 112 – unnecessary cast
return response.data as GooglePlaySubscriptionResponse;
```

### Finding 2: Custom `GooglePlayProductPurchaseResponse` duplicates `androidpublisher_v3.Schema$ProductPurchase`

- **File**: `apps/api/src/application/billing/services/google-play.service.ts:35`
- **Category**: sdk-type-duplication
- **Impact**: medium
- **Description**: `GooglePlayProductPurchaseResponse` at line 35 is a hand-rolled subset of the SDK's `Schema$ProductPurchase`. All five fields used (`acknowledgementState`, `consumptionState`, `obfuscatedExternalAccountId`, `orderId`, `purchaseState`, `purchaseTimeMillis`) are present with identical types in the SDK type. The cast at line 169 (`return response.data as GooglePlayProductPurchaseResponse`) is also unnecessary since the SDK already types `purchases.products.get` to return `Schema$ProductPurchase`.
- **Suggestion**: Remove `GooglePlayProductPurchaseResponse` and use `androidpublisher_v3.Schema$ProductPurchase` directly. Drop the `as` cast at line 169.
- **Evidence**:

```typescript
// google-play.service.ts:35-43
interface GooglePlayProductPurchaseResponse {
  acknowledgementState?: number;
  consumptionState?: number;
  obfuscatedExternalAccountId?: string;
  obfuscatedExternalProfileId?: string;
  orderId?: string;
  purchaseState?: number;
  purchaseTimeMillis?: string;
}

// line 169 – unnecessary cast
return response.data as GooglePlayProductPurchaseResponse;
```

### Finding 3: `ServiceAccountPayload.credentials` typed as `Record<string, unknown>` instead of the parsed JSON object type

- **File**: `apps/api/src/application/billing/services/google-play.service.ts:45`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `ServiceAccountPayload.credentials` is `Record<string, unknown>`. It is then passed directly into `google.auth.GoogleAuth({ credentials: ... })`. The `googleapis` `GoogleAuth` constructor accepts `JWTInput | ExternalAccountClientOptions | ...` (from `google-auth-library`). Using `Record<string, unknown>` means TypeScript cannot catch mismatches between the parsed JSON and what `GoogleAuth` actually requires. The `serviceAccountSchema` (line 50) only validates `client_email`, so the full credential object is passed as a wide record rather than a narrower type.
- **Suggestion**: Import `google-auth-library`'s `JWTInput` type (or `ServiceAccountCredentials`) and cast `parsed.value` to it after schema validation, or at minimum type `credentials` as `JWTInput` so `GoogleAuth`'s type parameter is satisfied without widening.
- **Evidence**:

```typescript
// google-play.service.ts:45-48
interface ServiceAccountPayload {
  credentials: Record<string, unknown>;
  email?: string;
}

// line 99 – passed to GoogleAuth which expects JWTInput
const auth = new google.auth.GoogleAuth({
  credentials: credentialsResult.value.credentials,
  scopes: ["https://www.googleapis.com/auth/androidpublisher"],
});
```

### Finding 4: Unsafe `as` cast for `autoRefillPackId` without type guard

- **File**: `apps/api/src/application/billing/services/billing.service.ts:486`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `user.autoRefillPackId` is `string | null` (bare `text()` column in Drizzle, no enum constraint). It is cast directly with `as "actions_1000" | "actions_5000" | "actions_15000"` (equivalently `as ActionPackId`) without any runtime narrowing. If the database contains a stale or invalid value, the cast silently produces an invalid `ActionPackId` that will propagate to callers expecting a valid pack ID. `ActionPackId` is available from `@slopweaver/contracts` and `actionPackIdSchema` from `@slopweaver/contracts` can be used for safe parsing.
- **Suggestion**: Replace the cast with a Zod parse or inline type guard using `actionPackIdSchema` from `@slopweaver/contracts`:
  ```typescript
  import { actionPackIdSchema } from "@slopweaver/contracts";
  const parsedPackId = actionPackIdSchema.safeParse(user.autoRefillPackId);
  autoRefillPackId: parsedPackId.success ? parsedPackId.data : null,
  ```
- **Evidence**:

```typescript
// billing.service.ts:486
autoRefillPackId: (user.autoRefillPackId as "actions_1000" | "actions_5000" | "actions_15000") ?? null,
```

### Finding 5: `as Checkout` and `as Subscription` casts in lemon-squeezy.service.ts bypass SDK type safety

- **File**: `apps/api/src/application/billing/services/lemon-squeezy.service.ts:346,597`
- **Category**: type-cast
- **Impact**: medium
- **Description**: Two `response.json()` results are cast to SDK types using `as Checkout` (line 346) and `as Subscription` (line 597). `response.json()` returns `unknown` (or `any` depending on fetch typings), so these casts bypass the TypeScript compiler's ability to detect SDK shape mismatches. The SDK types `Checkout` and `Subscription` from `@lemonsqueezy/lemonsqueezy.js` are already imported at the top of the file, but there is no runtime validation before the cast. If the API returns an unexpected shape (e.g. a change in API version), accessing `data.data.attributes.url` (line 347) will throw at runtime without a meaningful error.
- **Suggestion**: Add a runtime shape check or use `z.object(...)` to parse the response before accessing nested properties. At minimum, add an optional-chain (`data?.data?.attributes?.url`) rather than assuming the shape. For a more robust fix, define a Zod schema for the relevant subset of `Checkout`/`Subscription` attributes.
- **Evidence**:

```typescript
// lemon-squeezy.service.ts:346-347
const data = (await response.json()) as Checkout;
const checkoutUrl = data.data.attributes.url;

// lemon-squeezy.service.ts:597
const data = (await response.json()) as Subscription;
```
