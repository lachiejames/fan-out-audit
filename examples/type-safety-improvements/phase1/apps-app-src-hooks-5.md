# Type-Safety Audit: apps-app-src-hooks-5

Files audited:

- `apps/app/src/hooks/useInboxData.ts`
- `apps/app/src/hooks/useInboxItemActions.ts`
- `apps/app/src/hooks/useIntegrationHub.ts`
- `apps/app/src/hooks/useIntegrations.ts`
- `apps/app/src/hooks/useIntegrationsDirectory.ts`

---

## Finding 1

**File:** `apps/app/src/hooks/useInboxData.ts` (line 44)

**Category:** `Record<string, ...>` where stricter types would be safer

**Impact:** Low

**Description:**
The `InboxMessageMetadata` type includes a `Record<string, unknown>` catch-all variant for platform-specific metadata. While a catch-all is sometimes necessary for extensibility, it means consumers can't distinguish between platform-specific metadata shapes without runtime narrowing.

**Suggestion:**
Consider replacing the catch-all with a discriminated union keyed by platform ID (e.g., `{ platform: 'google-gmail'; threadId: string } | { platform: 'slack'; channelId: string } | ...`). If full coverage isn't feasible now, at minimum narrow the type to `Record<string, string | number | boolean | null>` to prevent deeply nested unknown structures.

**Evidence:**

```typescript
// InboxMessageMetadata union member
Record<string, unknown>;
```

---

## Finding 2

**File:** `apps/app/src/hooks/useInboxItemActions.ts` (lines 110, 191, 273, 365, 453)

**Category:** `any` or unsafe `unknown` in production code

**Impact:** Medium

**Description:**
Five `onMutate` handlers snapshot the query cache for optimistic rollback, but the snapshot type includes `counts: unknown` rather than the actual `MessageCounts` shape. This means rollback code has to cast or ignore the `counts` field.

**Suggestion:**
Explicitly type the snapshot with the cache shape interface (e.g., `{ counts: MessageCounts; messages: InboxMessage[]; total: number }`). The `MessageCounts` type is already defined in `useInboxData.ts` and should be imported here.

**Evidence:**

```typescript
// Five occurrences across mutation handlers
const snapshot = queryClient.getQueryData(...);  // inferred as: { counts: unknown; ... }
```

---

## Finding 3

**File:** `apps/app/src/hooks/useIntegrationHub.ts` (line 53)

**Category:** `Record<string, ...>` where stricter types would be safer

**Impact:** Low

**Description:**
`CATEGORY_TITLES` is typed as `Record<string, string>` but the keys are `KnowledgeCategory` values — a finite union type. Using `Record<string, string>` permits any string as a key, which means accessing `CATEGORY_TITLES['nonExistentCategory']` won't produce a TypeScript error.

**Suggestion:**
Change the type to `Record<KnowledgeCategory, string>` (imported from `@slopweaver/contracts`). This ensures exhaustive coverage at compile time: if a new `KnowledgeCategory` is added, TypeScript will require a corresponding entry in `CATEGORY_TITLES`.

**Evidence:**

```typescript
const CATEGORY_TITLES: Record<string, string> = { ... }
```

---

## Finding 4

**File:** `apps/app/src/hooks/useIntegrationHub.ts` (line 175)

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Low

**Description:**
`fact.category` is cast to `LearningEventType`. If `fact.category` is already typed as a string in the response body, the underlying type mismatch should be resolved at the source (in the contract schema or the transform function) rather than silently cast at the use site.

**Suggestion:**
If `fact.category` comes from the API, ensure the contract defines it as `LearningEventType`. Then no cast is needed. If the API returns a broader string, add a type guard instead of a cast.

**Evidence:**

```typescript
fact.category as LearningEventType;
```

---

## Finding 5

**File:** `apps/app/src/hooks/useIntegrations.ts` (line 93)

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Low

**Description:**
`item.platformMetadata` is cast to `Integration["platformMetadata"]`. This suggests that the raw API response type for the platform metadata field differs from the `Integration` type used in the UI — likely because the API response is typed more loosely. The cast hides this mismatch.

**Suggestion:**
Ensure the contract response type for `platformMetadata` matches the `Integration["platformMetadata"]` shape exactly. If the shapes differ, write an explicit transform function rather than using a cast.

**Evidence:**

```typescript
item.platformMetadata as Integration["platformMetadata"];
```

---

## Finding 6

**File:** `apps/app/src/hooks/useIntegrationsDirectory.ts` (line 45)

**Category:** Duplicate type definitions

**Impact:** High

**Description:**
`BackendIntegration` is defined as a local interface in `useIntegrationsDirectory.ts`. Identical (or near-identical) definitions exist in at least two other files: `useIntegrationHub.ts` and `usePlatformSync.ts`. Three separate definitions of the same API response shape will silently diverge when the backend changes.

**Suggestion:**
Extract `BackendIntegration` into a single shared location — either `@slopweaver/contracts` (preferred, as it derives from the API schema) or a shared type file at `apps/app/src/types/integration.ts`. Delete the three local copies and import the canonical type everywhere.

**Evidence:**

```typescript
// useIntegrationsDirectory.ts line ~45
interface BackendIntegration { ... }
// Also defined identically in useIntegrationHub.ts and usePlatformSync.ts
```
