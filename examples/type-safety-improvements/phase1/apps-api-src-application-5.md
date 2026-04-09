# Audit: apps-api-src-application-5

**Files inspected**: 8
**Findings**: 4

## Summary

These files cover the auth subsystem (Apple/Google OAuth services, the core AuthService, and auth helper utilities) plus the behavioral fingerprint feature (errors, repository, aggregator, and service). The auth files are generally well-typed. The main issues are a locally-defined `CookieOptions` interface that duplicates the one Express already exports, a `UserResponse` interface that duplicates a shape already defined in the contracts package, and a pair of identical handoff-record types across Apple and Google OAuth services. The behavioral files are clean with no significant type-safety issues.

## Findings

### Finding 1: `CookieOptions` duplicates Express's own exported interface

- **File**: `apps/api/src/application/auth/utils/auth-helpers.utils.ts:217`
- **Category**: sdk-type-duplication
- **Impact**: medium
- **Description**: A local `CookieOptions` interface is declared with fields `httpOnly`, `maxAge`, `path`, `sameSite`, and `secure` — all of which are already present on the `CookieOptions` interface exported from `@types/express-serve-static-core` (re-exported via `express`). The local interface uses required (`boolean`/`number`/`string`) fields where Express uses optional fields, meaning the local type is effectively a stricter subset of the SDK type. Any time the return value of the three `build*CookieOptions` functions is passed to `res.cookie()`, TypeScript must structurally reconcile two separate, unrelated interfaces. Importing the Express type directly removes the duplication and makes the relationship explicit.
- **Suggestion**: Remove the local interface and replace with:
  ```typescript
  import type { CookieOptions } from "express";
  ```
  Then annotate return types as `CookieOptions`. Because Express's interface uses `?` on all fields, no callers need to change; the return objects already satisfy it.
- **Evidence**:

```typescript
// auth-helpers.utils.ts:217
export interface CookieOptions {
  httpOnly: boolean;
  maxAge: number;
  path: string;
  sameSite: "lax" | "none" | "strict";
  secure: boolean;
}
```

---

### Finding 2: `UserResponse` duplicates the contract's `me` endpoint schema

- **File**: `apps/api/src/application/auth/services/auth.service.ts:55`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: A local `UserResponse` interface (`{ id: string; email: string; createdAt: string }`) is defined with a JSDoc comment saying it is "derived from baseAuthSchema.pick()". The contracts package already defines exactly this shape in `packages/contracts/src/contracts/auth/me.ts` via `baseAuthSchema.pick({ createdAt, email, id })`, which is the response schema for `GET /auth/me`. The `getUserById` return type and the `GET /auth/me` response schema are therefore two separate definitions of the same shape — if one diverges (e.g. a field is renamed in contracts), the service return type silently drifts.
- **Suggestion**: Export the inferred type from the contracts schema and import it here:

  ```typescript
  // In packages/contracts/src/contracts/auth/me.ts (or index)
  export type MeResponse = z.infer<typeof userResponseSchema>;

  // In auth.service.ts — replace the local interface:
  import type { MeResponse } from "@slopweaver/contracts";
  // ...
  async getUserById({ userId }: { userId: string }): Promise<Result<MeResponse, AuthError>>
  ```

  This makes the service return type the single source of truth, guaranteed to match the API contract.

- **Evidence**:

```typescript
// auth.service.ts:52-59
/**
 * Response type for getUserById - derived from baseAuthSchema.pick()
 * Matches the me endpoint response
 */
interface UserResponse {
  id: string;
  email: string;
  createdAt: string;
}

// packages/contracts/src/contracts/auth/me.ts:25-29 — same shape already defined:
const userResponseSchema = baseAuthSchema.pick({
  createdAt: true,
  email: true,
  id: true,
});
```

---

### Finding 3: `AppleAuthHandoffRecord` and `GoogleAuthHandoffRecord` are identical duplicate types

- **File**: `apps/api/src/application/auth/services/apple-oauth.service.ts:52` and `apps/api/src/application/auth/services/google-oauth.service.ts:52`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: Both OAuth services define a private local type with the exact same three fields:
  ```
  { accessToken: string; expiresAt: number; refreshToken: string }
  ```
  These types are structurally identical but nominally different, so any future divergence (e.g. adding a `tokenType` field to one) would require remembering to update both. They are used symmetrically: stored in cache, returned from `consumeHandoff`, and passed to `createNativeHandoff`. Because `AuthService.setAuthCookies` also accepts `{ accessToken, refreshToken, expiresAt }`, there is already an implicit shared shape that both services satisfy.
- **Suggestion**: Extract a shared `AuthHandoffRecord` type into the auth utilities file or a shared auth types file, and import it in both services:
  ```typescript
  // In auth-helpers.utils.ts or a new auth.types.ts
  export type AuthHandoffRecord = {
    accessToken: string;
    expiresAt: number;
    refreshToken: string;
  };
  ```
  Both `AppleOAuthService` and `GoogleOAuthService` then import and use `AuthHandoffRecord` instead of their own copies.
- **Evidence**:

```typescript
// apple-oauth.service.ts:52-56
type AppleAuthHandoffRecord = {
  accessToken: string;
  expiresAt: number;
  refreshToken: string;
};

// google-oauth.service.ts:52-56  (byte-for-byte identical)
type GoogleAuthHandoffRecord = {
  accessToken: string;
  expiresAt: number;
  refreshToken: string;
};
```

---

### Finding 4: `enrichWithDisplayNames` return type silently widens `formalityByRecipient` / `responseSpeedByRecipient` to `Record<string, number>`

- **File**: `apps/api/src/application/behavioral/services/behavioral.service.ts:383`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The private `enrichWithDisplayNames` method builds `enrichedFormality` and `enrichedSpeed` as `Record<string, number>`, then spreads them back into the `BehavioralFingerprint` object. However the table type for these fields (imported from `behavioral-fingerprint.table`) is the narrower `FormalityByRecipient` / `ResponseSpeedByRecipient` branded or aliased types. The spread `{ ...fingerprint, formalityByRecipient: enrichedFormality, responseSpeedByRecipient: enrichedSpeed }` effectively widens the return type back to `BehavioralFingerprint` only because `Record<string, number>` is structurally compatible — but if those table types are ever narrowed (e.g. to a branded type or a stricter mapped type), this silent widening will break at runtime before the compiler catches it. The accumulator variables should be typed to match the table column types, not the generic `Record<string, number>`.
- **Suggestion**: Type the accumulator variables using the imported column types:

  ```typescript
  import type { FormalityByRecipient, ResponseSpeedByRecipient } from "@/shared/db/tables/behavioral-fingerprint.table";

  const enrichedFormality: FormalityByRecipient = {};
  const enrichedSpeed: ResponseSpeedByRecipient = {};
  ```

  This ensures the enrichment logic stays in sync with the table schema types automatically.

- **Evidence**:

```typescript
// behavioral.service.ts:383-394
const enrichedFormality: Record<string, number> = {};
for (const [entityId, score] of Object.entries(fingerprint.formalityByRecipient ?? {})) {
  const displayName = entityIdToName.get(entityId) ?? entityId;
  enrichedFormality[displayName] = score;
}

const enrichedSpeed: Record<string, number> = {};
for (const [entityId, speed] of Object.entries(fingerprint.responseSpeedByRecipient ?? {})) {
  const displayName = entityIdToName.get(entityId) ?? entityId;
  enrichedSpeed[displayName] = speed;
}

return {
  ...fingerprint,
  formalityByRecipient: enrichedFormality, // widened to Record<string,number>
  responseSpeedByRecipient: enrichedSpeed, // widened to Record<string,number>
};
```
