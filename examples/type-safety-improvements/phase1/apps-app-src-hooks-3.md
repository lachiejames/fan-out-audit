# Type-Safety Audit: apps-app-src-hooks-3

Files audited:

- `apps/app/src/hooks/useAnalyticsData.ts`
- `apps/app/src/hooks/useBehavioralFingerprint.ts`
- `apps/app/src/hooks/useBulkActions.ts`
- `apps/app/src/hooks/useChatPersistence.ts`

---

## Finding 1

**File:** `apps/app/src/hooks/useAnalyticsData.ts`

**Category:** Custom types duplicating SDK/contract types

**Impact:** Medium

**Description:**
The file defines four local interfaces — `PredictionStatsResponse`, `TrainingStatsResponse`, `ChatFeedbackStatsResponse`, and `TriageOverrideStatsResponse` — that describe API response shapes. These are likely duplicates of (or should be derived from) types already exported by `@slopweaver/contracts`. If the contract types ever change, these local copies will silently drift.

**Suggestion:**
Check whether `@slopweaver/contracts` already exports analytics response types. If so, import them directly and delete the local definitions. If not, add them to the contracts package and import from there.

**Evidence:**

```typescript
// useAnalyticsData.ts — local interface definitions
interface PredictionStatsResponse { ... }
interface TrainingStatsResponse { ... }
interface ChatFeedbackStatsResponse { ... }
interface TriageOverrideStatsResponse { ... }
```

---

## Finding 2

**File:** `apps/app/src/hooks/useBehavioralFingerprint.ts` (line 82)

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Low

**Description:**
The non-200 error body is cast to `{ message?: string }` to extract an error message. This is a speculative cast — the actual body shape at non-200 status codes is determined by the backend's standard error schema but is not narrowed by ts-rest.

**Suggestion:**
Use `extractErrorMessage` from `@slopweaver/contracts`, which already handles the standard error body shape safely without requiring a cast.

**Evidence:**

```typescript
const errorBody = response.body as { message?: string };
```

---

## Finding 3

**File:** `apps/app/src/hooks/useBulkActions.ts` (line 163)

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Low

**Description:**
Same pattern as Finding 2 — the non-200 body is cast to `{ error?: string }` to extract an error string. The field name (`error` instead of `message`) does not align with the standard error schema.

**Suggestion:**
Use `extractErrorMessage` from `@slopweaver/contracts` for a consistent, type-safe extraction path.

**Evidence:**

```typescript
const errorBody = response.body as { error?: string };
```

---

## Finding 4

**File:** `apps/app/src/hooks/useChatPersistence.ts` (line 22)

**Category:** `any` or unsafe `unknown` in production code

**Impact:** Medium

**Description:**
The hook accepts `user?: unknown` as a parameter. Downstream code must narrow this parameter before any property access, but no narrowing is visible at the call site. Using `unknown` as a param type for something that is clearly a user object invites unsafe access patterns and obscures the intended contract.

**Suggestion:**
Type the `user` parameter explicitly — at minimum as `{ id: string } | null | undefined`, or ideally as the full user type from the auth context (e.g. `import type { User } from '@/lib/auth-context'`).

**Evidence:**

```typescript
// useChatPersistence.ts line 22
user?: unknown
```

---

## Finding 5

**File:** `apps/app/src/hooks/useChatPersistence.ts` (lines 76–77)

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Low

**Description:**
A message object literal is cast to `UIMessage` from the `ai` SDK. This cast forces an unverified object into a type without compile-time validation of required fields — if the SDK's `UIMessage` type gains new required fields, this cast will hide the mismatch.

**Suggestion:**
Build the object with a `satisfies UIMessage` assertion instead of `as UIMessage`, or import and use the SDK's constructor utilities if available. This surfaces missing required fields at compile time rather than hiding them.

**Evidence:**

```typescript
} as UIMessage
```
