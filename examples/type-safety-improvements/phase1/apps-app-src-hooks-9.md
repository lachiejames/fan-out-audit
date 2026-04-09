# Type-Safety Audit: apps-app-src-hooks-9

Files audited:

- `apps/app/src/hooks/useSearchData.ts`
- `apps/app/src/hooks/useSessionIdleDetection.ts`
- `apps/app/src/hooks/useSettingsData.ts`
- `apps/app/src/hooks/useSuggestedActionExecution.ts`
- `apps/app/src/hooks/useSuggestedActions.ts`
- `apps/app/src/hooks/useSuggestionFeedback.ts`
- `apps/app/src/hooks/useSyncStream.ts`
- `apps/app/src/hooks/useTauriEvents.ts`

---

## Finding 1

**File:** `apps/app/src/hooks/useSearchData.ts`

**Category:** `Record<string, ...>` where stricter types would be safer

**Impact:** Low

**Description:**
`platformCounts: Record<string, number>` uses unconstrained string keys for platform-keyed count data. The same issue appears in `useQueueData.ts` (see `apps-app-src-hooks-8.md` Finding 5).

**Suggestion:**
Change to `Partial<Record<PlatformId, number>>` (importing `PlatformId` from `@slopweaver/contracts`). This prevents typos and enables exhaustive handling.

**Evidence:**

```typescript
platformCounts: Record<string, number>;
```

---

## Finding 2

**File:** `apps/app/src/hooks/useSettingsData.ts`

**Category:** Custom types duplicating SDK/contract types

**Impact:** High

**Description:**
Two interfaces — `UserSettings` and `UpdateSettingsInput` — are defined locally. Settings types of this kind are almost always defined in the contracts package because they mirror the API's GET/PATCH schemas for the settings endpoint. If contract types already exist, these local definitions are duplicates.

**Suggestion:**
Import `UserSettings` (or the equivalent response type) and `UpdateSettingsInput` (or the patch body type) from `@slopweaver/contracts`. Delete the local definitions. If the UI requires a subset or extension, use `Pick<UserSettings, ...>` or intersection types rather than a full re-declaration.

**Evidence:**

```typescript
interface UserSettings { ... }       // local definition
interface UpdateSettingsInput { ... } // local definition
```

---

## Finding 3

**File:** `apps/app/src/hooks/useSettingsData.ts`

**Category:** `any` or unsafe `unknown` in production code

**Impact:** Medium

**Description:**
`updateSettingsAsync: (...) => Promise<unknown>` is the return type for the async mutation method. Callers cannot safely inspect the result without a cast.

**Suggestion:**
Derive the return type from the contract's 200 response body: `Promise<Awaited<ReturnType<typeof tsr.settings.updateSettings.mutate>>['body']>` (conditioned on status 200), or use the contract schema type directly.

**Evidence:**

```typescript
updateSettingsAsync: (...) => Promise<unknown>
```

---

## Finding 4

**File:** `apps/app/src/hooks/useSyncStream.ts` (line 173)

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Medium

**Description:**
`const body = startResponse.body as unknown` followed by destructuring as `Record<string, unknown>` with additional field casts is a multi-step unsafe pattern. The sync stream start endpoint's body is a known shape; typing it as `unknown` and then re-casting each field is error-prone.

**Suggestion:**
Type the `startResponse.body` using the contract's defined 200 response schema for the sync-stream start endpoint. With a correctly-typed contract, no intermediate `as unknown` or `Record<string, unknown>` casts are needed.

**Evidence:**

```typescript
const body = startResponse.body as unknown;
// then later:
const { streamId, ... } = body as Record<string, unknown>;
```

---

## Finding 5

**File:** `apps/app/src/hooks/useSuggestedActions.ts`

No type-safety issues found. This file correctly imports response types from `@slopweaver/contracts` (`SuggestedActionResponse`), uses `select: (d) => d.body` to unwrap the ts-rest envelope, and avoids inline casts.

---

## Finding 6

**File:** `apps/app/src/hooks/useSuggestedActionExecution.ts`

No significant type-safety issues found. The hook uses `extractErrorMessage` from `@slopweaver/contracts` for error body handling and derives return types from the mutation function's result. Minor: the error callback parameter is typed as `unknown` (standard for useMutation), which is acceptable.

---

## Finding 7

**File:** `apps/app/src/hooks/useSuggestionFeedback.ts`

No type-safety issues found. The file uses typed contract imports and avoids unsafe casts.

---

## Finding 8

**File:** `apps/app/src/hooks/useTauriEvents.ts`

No type-safety issues found. Event payloads from `@tauri-apps/api/event` are properly typed via the generic `listen<PayloadType>()` API, and all handler callbacks use the specific payload types.
