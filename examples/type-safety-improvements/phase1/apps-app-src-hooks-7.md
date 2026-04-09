# Type-Safety Audit: apps-app-src-hooks-7

Files audited:

- `apps/app/src/hooks/usePatternLearningProgress.ts`
- `apps/app/src/hooks/usePatternLearningStream.ts`
- `apps/app/src/hooks/usePendingIntegration.ts`
- `apps/app/src/hooks/usePlatformSync.ts`

---

## Finding 1

**File:** `apps/app/src/hooks/usePatternLearningProgress.ts`

**Category:** Custom types duplicating SDK/contract types

**Impact:** Medium

**Description:**
The file defines a local `PatternLearningProgress` interface with fields `pendingEdits`, `extractionThreshold`, and `editsUntilNextExtraction`. These fields closely match data that should be driven by the pattern-extraction domain in the contracts package. If this type already exists in `@slopweaver/contracts`, the local definition is a duplicate that can silently drift.

**Suggestion:**
Check whether `@slopweaver/contracts` exports a type for pattern learning progress. If it does, delete the local interface and import the canonical type. If not, add it to the contracts package.

**Evidence:**

```typescript
interface PatternLearningProgress {
  pendingEdits: number;
  extractionThreshold: number;
  editsUntilNextExtraction: number;
  // ...
}
```

---

## Finding 2

**File:** `apps/app/src/hooks/usePatternLearningStream.ts` (lines 65–66)

**Category:** `Record<string, ...>` where stricter types would be safer

**Impact:** Medium

**Description:**
`event.metadata` is typed as `Record<string, unknown>` on `ActivityStreamEvent`. The code accesses `event.metadata["extractionMode"]` and `event.metadata["patternsExtracted"]` by bracket notation, which returns `unknown` and then requires downstream type assertions. The two specific fields being accessed are known at compile time.

**Suggestion:**
Either add `extractionMode` and `patternsExtracted` as explicit optional fields on the `ActivityStreamEvent.metadata` type in `@slopweaver/contracts`, or create a discriminated union for the specific stream event shapes that carry these fields. This eliminates bracket-access on `unknown` values.

**Evidence:**

```typescript
event.metadata["extractionMode"];
event.metadata["patternsExtracted"];
```

---

## Finding 3

**File:** `apps/app/src/hooks/usePatternLearningStream.ts` (line 79)

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Low

**Description:**
`(p as Record<string, unknown>)["type"]` casts a pattern object to access its `type` field. This is a workaround for the pattern object not having an explicit `type` field in its declared type.

**Suggestion:**
Add a `type` field to the pattern object type in the contract schema. This eliminates the cast entirely.

**Evidence:**

```typescript
(p as Record<string, unknown>)["type"];
```

---

## Finding 4

**File:** `apps/app/src/hooks/usePendingIntegration.ts` (line 136)

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Low

**Description:**
`setConfirmedIntegration(integration as ConfirmedIntegration)` casts a broader `Integration` type (or similar) to `ConfirmedIntegration`. If the runtime object genuinely satisfies `ConfirmedIntegration`, this should be verified with a type guard rather than assumed via cast.

**Suggestion:**
Write a `isConfirmedIntegration(value: unknown): value is ConfirmedIntegration` type predicate and use it before the call to `setConfirmedIntegration`. This validates the shape at runtime and removes the unsafe cast.

**Evidence:**

```typescript
setConfirmedIntegration(integration as ConfirmedIntegration);
```

---

## Finding 5

**File:** `apps/app/src/hooks/usePendingIntegration.ts` (line 172)

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Medium

**Description:**
`confirmMutation.mutateAsync as UsePendingIntegrationReturn["confirmConnection"]` casts the mutation's async function to match the declared return interface. This is typically a sign that the mutation's inferred return type doesn't exactly match the interface declaration — the cast silences the mismatch rather than fixing it.

**Suggestion:**
Align the `mutationFn` return type with the declared interface, or derive the interface type directly from the mutation return type using `Awaited<ReturnType<typeof confirmMutation.mutateAsync>>`.

**Evidence:**

```typescript
confirmMutation.mutateAsync as UsePendingIntegrationReturn["confirmConnection"];
```

---

## Finding 6

**File:** `apps/app/src/hooks/usePlatformSync.ts` (line 89)

**Category:** Duplicate type definitions

**Impact:** High

**Description:**
`BackendIntegration` is defined for the third time (also in `useIntegrationHub.ts` and `useIntegrationsDirectory.ts`). See Finding 6 in `apps-app-src-hooks-5.md`. This confirms the pattern: three hooks independently define the same interface.

**Suggestion:**
Consolidate into a single definition in `@slopweaver/contracts` or `apps/app/src/types/integration.ts`. Delete all three local copies.

**Evidence:**

```typescript
// usePlatformSync.ts line ~89
interface BackendIntegration { ... }
```

---

## Finding 7

**File:** `apps/app/src/hooks/usePlatformSync.ts` (line 318)

**Category:** Custom types duplicating SDK/contract types

**Impact:** Low

**Description:**
An inline `{ syncInProgress: boolean }` annotation is used instead of importing the contract's sync status type. This is a micro-duplication but contributes to type drift if the sync status shape changes.

**Suggestion:**
Import the sync status type from `@slopweaver/contracts` and use it directly.

**Evidence:**

```typescript
{
  syncInProgress: boolean;
} // inline instead of using the contract type
```
