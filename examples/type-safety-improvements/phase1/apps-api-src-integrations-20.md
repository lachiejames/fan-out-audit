# Type Safety Audit — apps/api/src/integrations (Batch 20)

**Files inspected**: 8  
**Findings**: 3

## Summary

The Gmail/Google shared utilities are generally well-typed, leveraging `gmail_v1` SDK types throughout. Three issues were found: an unnecessary type cast in `google-oauth.utils.ts` (the wrapped function already returns `Date`), an `[key: string]: unknown` index signature on `GooglePlatformMetadata` that prevents safe property access in subclass code, and a `platformMetadata as TPlatformMetadata | null` cast in the OAuth base service that is structurally unavoidable but hides the fact that the stored value is `unknown`.

## Findings

---

### Finding 1

**File**: `apps/api/src/integrations/platforms/google/google-oauth.utils.ts:75`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `calculateTokenExpiry` casts the return of `_calculateTokenExpiry` to `Date` even though `_calculateTokenExpiry` already returns `Date`. The cast is unnecessary and misleading — it implies `_calculateTokenExpiry` might not return `Date`.

**Suggestion**: Remove the `as Date` cast. The return type of `_calculateTokenExpiry` is already `Date`, so no cast is needed.

**Evidence**:

```typescript
// google-oauth.utils.ts:75
export function calculateTokenExpiry({
  expiresIn,
  now = Date.now(),
}: {
  expiresIn: number;
  now?: number | undefined;
}): Date {
  return _calculateTokenExpiry({ expiresIn, now }) as Date; // unnecessary cast
}
```

---

### Finding 2

**File**: `apps/api/src/integrations/platforms/google/google-oauth-base.service.ts:81`  
**Category**: `Record<string, ...>` weakening  
**Impact**: Low

**Description**: `GooglePlatformMetadata` declares an open `[key: string]: unknown` index signature. This prevents TypeScript from enforcing that access to known properties goes through the typed fields, and any arbitrary string key access silently returns `unknown` instead of causing a compile error.

**Suggestion**: Remove the index signature from `GooglePlatformMetadata`. Platform-specific subclass types (e.g., `GmailPlatformMetadata`) should declare all fields they actually use as named properties without falling back to a catch-all.

**Evidence**:

```typescript
// google-oauth-base.service.ts:79-82
export type GooglePlatformMetadata = PlatformMetadata & {
  email: string;
  [key: string]: unknown; // prevents exhaustive property checking
};
```

---

### Finding 3

**File**: `apps/api/src/integrations/platforms/google/google-oauth-base.service.ts:398`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `buildMasterUserInfo` receives `integration.platformMetadata` typed as `unknown` (from the database) and casts it directly to the generic `TPlatformMetadata | null` without any runtime validation. If the stored JSON is malformed or from an older schema, the cast will succeed silently and subsequent property access will produce runtime errors.

**Suggestion**: Parse the metadata through a Zod schema before casting, or use a type guard that validates the required `email` field exists.

**Evidence**:

```typescript
// google-oauth-base.service.ts:397-398
protected buildMasterUserInfo(integration: { platformUserId: string; platformMetadata: unknown; }): GoogleUserInfoResponse {
  const metadata = integration.platformMetadata as TPlatformMetadata | null;  // unsafe cast from unknown
```
