# Type-Safety Audit: apps-app-src-hooks-8

Files audited:

- `apps/app/src/hooks/usePriorityScoring.ts`
- `apps/app/src/hooks/useProactivePreferences.ts`
- `apps/app/src/hooks/useQueueData.ts`
- `apps/app/src/hooks/useQueueFilters.ts`

---

## Finding 1

**File:** `apps/app/src/hooks/usePriorityScoring.ts` (lines 116, 139)

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Low

**Description:**
`response.body as VipContact` is used twice to extract the VIP contact from a non-discriminated ts-rest response body. The cast assumes the 200 response body is a `VipContact`, but if the contract defines the 200 body with the correct schema, no cast should be needed.

**Suggestion:**
Ensure the ts-rest contract for this endpoint defines the 200 response as `vipContactResponseSchema` (the Zod schema for `VipContact`). With a correctly-typed contract and `select: (d) => d.body`, the type flows automatically.

**Evidence:**

```typescript
response.body as VipContact; // line 116
response.body as VipContact; // line 139
```

---

## Finding 2

**File:** `apps/app/src/hooks/usePriorityScoring.ts`

**Category:** Custom types duplicating SDK/contract types

**Impact:** Medium

**Description:**
A local `VipContact` interface is defined in this file. `VipContact` is almost certainly part of the contacts/priority-scoring domain in `@slopweaver/contracts`. If both definitions exist, they will silently diverge.

**Suggestion:**
Import `VipContact` from `@slopweaver/contracts` and remove the local definition.

**Evidence:**

```typescript
interface VipContact { ... }  // local definition — likely duplicates contracts type
```

---

## Finding 3

**File:** `apps/app/src/hooks/useProactivePreferences.ts`

**Category:** Custom types duplicating SDK/contract types

**Impact:** Medium

**Description:**
`ProactivePreferences` is a local interface that manually maps fields from `ProactiveSettings` (which comes from the API). If `ProactiveSettings` is already exported from `@slopweaver/contracts`, then `ProactivePreferences` is either a rename or a subset — both cases are better handled by using (or extending) the contract type directly.

**Suggestion:**
Check if `ProactiveSettings` from `@slopweaver/contracts` already covers all the required fields. If it does, remove `ProactivePreferences` and use `ProactiveSettings` directly (or as `Partial<ProactiveSettings>` where appropriate). If UI-only fields need to be added, use an intersection type rather than a full re-declaration.

**Evidence:**

```typescript
// Local interface — manual mapping of ProactiveSettings
interface ProactivePreferences {
  // fields that mirror ProactiveSettings from @slopweaver/contracts
}
```

---

## Finding 4

**File:** `apps/app/src/hooks/useQueueData.ts`

**Category:** `any` or unsafe `unknown` in production code

**Impact:** Medium

**Description:**
`editWorkItem` and `undoSend` return `Promise<unknown>`. Callers that need to inspect the resolved value (e.g., to extract an undo deadline) have no type-safe path.

**Suggestion:**
Derive explicit return types from the contract response schemas. `editWorkItem` should return the 200 body type from `tsr.workItems.updateWorkItem`; `undoSend` should return the 200 body from `tsr.workItems.undoWorkItem`.

**Evidence:**

```typescript
editWorkItem: (...) => Promise<unknown>
undoSend: (...) => Promise<unknown>
```

---

## Finding 5

**File:** `apps/app/src/hooks/useQueueData.ts`

**Category:** `Record<string, ...>` where stricter types would be safer

**Impact:** Low

**Description:**
`platformCounts: Record<string, number>` and `statusCounts: Record<string, number>` in the result interface use unconstrained string keys. `platformCounts` keys should be `PlatformId` values; `statusCounts` keys should be a known status union.

**Suggestion:**
Change `platformCounts` to `Partial<Record<PlatformId, number>>` (imported from `@slopweaver/contracts`) and `statusCounts` to `Partial<Record<WorkItemStatus, number>>`. This makes exhaustive handling possible and prevents typos in key names.

**Evidence:**

```typescript
platformCounts: Record<string, number>;
statusCounts: Record<string, number>;
```

---

## Finding 6

**File:** `apps/app/src/hooks/useQueueFilters.ts`

No type-safety issues found. This file is clean — it uses well-typed Zustand store patterns with explicit filter state types derived from the queue domain. No findings for this file.
