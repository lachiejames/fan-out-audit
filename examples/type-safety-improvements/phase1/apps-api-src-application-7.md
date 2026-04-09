# Audit: apps-api-src-application-7

**Files inspected**: 8
**Findings**: 5

## Summary

These files implement the billing domain: Redis usage stats utilities, NestJS decorator helpers, a discriminated-union error module, and four services covering Apple App Store JWS verification, App Store Server API calls, action-balance management, and account freeze/unfreeze. Overall the code is well-structured; the main issues are an unchecked `as T` cast in the JWS verification path, an `as AppStoreTransactionResponse` cast on a raw fetch response, a duplicate transaction-shape interface that overlaps with the already-exported Zod-inferred type, a fully-untyped `Record<string, unknown>` return for signed renewal payloads (when a Zod schema and exported type already exist for that data), and several `metadata` parameters typed as `Record<string, unknown>` that could be narrowed.

## Findings

### Finding 1: Unchecked `as T` cast after JWS verification

- **File**: `apps/api/src/application/billing/services/app-store-notification.service.ts:216`
- **Category**: type-cast
- **Impact**: high
- **Description**: `jwtVerify` (from `jose`) returns `payload` typed as `JWTPayload` — a well-known type. The code coerces it directly to the generic `T` with `payload as T`. If the caller passes a concrete type (e.g. `AppStoreServerNotificationPayload`) the cast bypasses all runtime shape validation. A malformed or tampered token payload would pass TypeScript's type checker and flow through as a trusted typed value. The proper fix is to run the caller-provided Zod schema through the same `decodeJwtPayload` helper (which already exists and does the schema parse), or alternatively parse `payload` through the relevant schema before returning.
- **Suggestion**: Return `payload` as `JWTPayload` (the jose type) and let each public method parse it through its own Zod schema, exactly as the test path already does via `decodeJwtPayload`. Import `JWTPayload` from `jose` to get the correct static type.
- **Evidence**:

```typescript
// app-store-notification.service.ts lines 213–219
try {
  const key = await importX509(leafCert.toString(), "ES256");
  const { payload } = await jwtVerify(jwsToken, key, { algorithms: ["ES256"] });
  return ok(payload as T); // <-- unsafe cast: T is caller-chosen, no schema validation
} catch (error) {
  return err(BillingErrors.invalidRequest("[APPLE] JWS signature verification failed"));
}
```

---

### Finding 2: Unvalidated `as AppStoreTransactionResponse` cast on fetch response

- **File**: `apps/api/src/application/billing/services/app-store.service.ts:91`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `response.json()` returns `unknown` (or `any` depending on lib.dom version). Casting it directly to `AppStoreTransactionResponse` with `as` gives false type safety — if Apple changes their response shape or returns an error JSON, the downstream destructuring (`transactionResponse.signedTransactionInfo`) silently gets `undefined` instead of raising a type error at the boundary. A Zod parse (or at minimum a type guard) would catch unexpected shapes.
- **Suggestion**: Define a `z.object({ signedTransactionInfo: z.string().optional(), signedRenewalInfo: z.string().optional() })` schema (matching `AppStoreTransactionResponse`) and parse the response through it, or reuse the existing `parseJson` utility already imported in the sibling notification service.
- **Evidence**:

```typescript
// app-store.service.ts lines 88–92
return (await response.json()) as AppStoreTransactionResponse;
```

---

### Finding 3: Duplicate Apple transaction shape — `AppleSignedTransaction` vs `AppStoreSignedTransaction`

- **File**: `apps/api/src/application/billing/services/app-store.service.ts:30`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `app-store.service.ts` defines a private `AppleSignedTransaction` interface (lines 30–37) describing `appAccountToken`, `expiresDate`, `originalTransactionId`, `productId`, `purchaseDate`, `transactionId`. The sibling `app-store-notification.service.ts` already exports `AppStoreSignedTransaction` (derived from a Zod schema, lines 38–53) with an identical field set. The `AppStoreService.buildTestTransactionInfo` method constructs a value typed as `AppleSignedTransaction` — it could type it as `AppStoreSignedTransaction` from the notification service instead, eliminating the duplicate. The discrepancy (`productId` is required in the private type but optional in the Zod-inferred one) is an additional latent inconsistency.
- **Suggestion**: Delete the private `AppleSignedTransaction` interface in `app-store.service.ts` and import `AppStoreSignedTransaction` from `app-store-notification.service.ts`. Align the `productId` optionality between the two files (the real Apple API makes it required, so the Zod schema should use `z.string()` not `z.string().optional()`).
- **Evidence**:

```typescript
// app-store.service.ts lines 30–37 (private duplicate)
interface AppleSignedTransaction {
  appAccountToken?: string;
  expiresDate?: string;
  originalTransactionId?: string;
  productId: string; // required here
  purchaseDate?: string;
  transactionId?: string;
}

// app-store-notification.service.ts lines 38–53 (canonical, exported)
const appStoreSignedTransactionSchema = z
  .object({
    appAccountToken: z.string().optional(),
    expiresDate: z.string().optional(),
    originalTransactionId: z.string().optional(),
    productId: z.string().optional(), // optional here — inconsistency
    purchaseDate: z.string().optional(),
    transactionId: z.string().optional(),
  })
  .loose();
export type AppStoreSignedTransaction = z.infer<typeof appStoreSignedTransactionSchema>;
```

---

### Finding 4: `verifyAndDecodeRenewal` returns `Record<string, unknown>` when a Zod schema already exists

- **File**: `apps/api/src/application/billing/services/app-store-notification.service.ts:159`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `verifyAndDecodeRenewal` returns `Result<Record<string, unknown>, BillingError>` and passes `Record<string, unknown>` as the generic argument to `verifyAndDecodeJws`. A private Zod schema `appStoreSignedRenewalSchema` is already defined on line 50 (`z.record(z.string(), z.unknown())`), but no exported type is derived from it and no shape narrowing occurs. If Apple's renewal payload has known fields the codebase already relies upon (e.g. `autoRenewStatus`, `expirationIntent`), this is an opportunity to define a stricter Zod schema. At minimum, an exported `AppStoreSignedRenewal` type should be derived via `z.infer` and used as the return type, parallel to how `AppStoreSignedTransaction` is exported.
- **Suggestion**: Either define concrete known fields on `appStoreSignedRenewalSchema` and export `type AppStoreSignedRenewal = z.infer<typeof appStoreSignedRenewalSchema>`, or — if the fields are truly unknown — document the intentional looseness. Replace the raw `Record<string, unknown>` in the return type with the named alias to improve call-site readability and enable future narrowing.
- **Evidence**:

```typescript
// line 50 — schema defined but no exported type
const appStoreSignedRenewalSchema = z.record(z.string(), z.unknown());

// lines 155–178 — return type uses raw Record instead of named alias
async verifyAndDecodeRenewal({ signedRenewalInfo }: { signedRenewalInfo: string; }):
  Promise<Result<Record<string, unknown>, BillingError>> {
  ...
  const payloadResult = await this.verifyAndDecodeJws<Record<string, unknown>>({ jwsToken: signedRenewalInfo });
```

---

### Finding 5: `metadata` parameters typed as `Record<string, unknown>` without narrowing

- **File**: `apps/api/src/application/billing/services/billing-actions.service.ts:175`, `:288`, `:690`
- **Category**: record-weakening
- **Impact**: low
- **Description**: Three public methods (`deductActions`, `addActions`, `revokeAllActions`) accept `metadata?: Record<string, unknown>`. The metadata is spread directly into transaction records persisted to the database. Because the type is fully open, callers can pass any shape without compile-time checking, and there is no guarantee that known fields (e.g. `units`, `previousBalance`, `idempotencyKey`) are present or correctly typed when they are expected. At insertion sites the code manually adds known keys (`units: normalizedUnits`) alongside the caller's open map, which means the actual database record has a mix of enforced and unvalidated fields. Narrowing to a discriminated union or a typed interface (e.g. `{ units?: number; idempotencyKey?: string; [key: string]: unknown }`) would at least anchor the known fields.
- **Suggestion**: Define a `BillingTransactionMetadata` interface with the known fields as optional typed properties plus an index signature for extensibility, and replace `Record<string, unknown>` at all three call sites. This prevents caller mistakes like passing a string where a number is expected for `units`.
- **Evidence**:

```typescript
// billing-actions.service.ts line 175
metadata?: Record<string, unknown> | undefined;

// line 237 — known field added alongside open metadata at insertion
metadata: metadata ? { ...metadata, units: normalizedUnits } : { units: normalizedUnits },

// line 288
metadata?: Record<string, unknown> | undefined;

// line 690
metadata?: Record<string, unknown>;
```
