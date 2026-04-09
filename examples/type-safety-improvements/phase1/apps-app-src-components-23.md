# Type-Safety Audit: apps-app-src-components-23

**Files audited:**

- `apps/app/src/components/settings/profile/settings-current-focus-section.tsx`
- `apps/app/src/components/settings/profile/settings-people-section.tsx`
- `apps/app/src/components/settings/profile/settings-profile-header.tsx`
- `apps/app/src/components/settings/profile/settings-voice-section.tsx`
- `apps/app/src/components/settings/settings-account.tsx`
- `apps/app/src/components/settings/settings-billing-content.tsx`
- `apps/app/src/components/settings/settings-billing.tsx`
- `apps/app/src/components/settings/settings-integrations.tsx`

---

## Findings

### 1. `platform` param typed as `string` instead of `PlatformId` — `settings-people-section.tsx` line 87

**Category:** Custom types duplicating or weaker than SDK/contract types

**Location:** `apps/app/src/components/settings/profile/settings-people-section.tsx:87`

**Description:**
The local helper `formatPlatformName({ platform }: { platform: string })` accepts a raw `string` parameter. The codebase has `PlatformId` from `@slopweaver/contracts` as the source of truth for valid platform identifiers. Passing any arbitrary string bypasses the exhaustiveness check and may silently produce a fallback label when a typo or unknown platform ID is passed.

**Suggested fix:**

```typescript
import type { PlatformId } from "@slopweaver/contracts";

function formatPlatformName({ platform }: { platform: PlatformId }): string {
  // ...
}
```

If the call sites supply values that may not be `PlatformId` (e.g., from user-entered data), add a runtime check at the boundary rather than weakening the internal type.

---

### 2. Positional parameter violation — `settings-integrations.tsx`

**Category:** (Project rule violation, not strictly a type-safety category)

**Location:** `apps/app/src/components/settings/settings-integrations.tsx`

**Description:**
The `handleDisconnect` function is defined with a positional `integrationId: string` parameter. Per the project's mandatory named-params rule, all functions with 1+ parameters must use named object params.

**Suggested fix:**

```typescript
// Before
function handleDisconnect(integrationId: string): void { ... }

// After
function handleDisconnect({ integrationId }: { integrationId: string }): void { ... }
```

---

### 3. Complex inline narrowing for step `name` key — `settings-integrations.tsx` line 301–302

**Category:** Avoidable type casts / unsafe narrowing

**Location:** `apps/app/src/components/settings/settings-integrations.tsx:301`

**Description:**
The render expression `key={typeof step === "object" && "name" in step ? step.name : index}` performs a complex inline type narrowing because the `step` union includes fallback objects that lack a `name` property. If the step type is a union of named types from contracts, consider making both variants carry a stable `id` or `key` field so the narrowing becomes unnecessary.

**Suggested fix:** Define a discriminated union for wizard steps with a shared `id` field, or ensure all step variants carry a `name`/`key` property so the key expression simplifies to `step.name ?? index`.

---

## Clean Files

- `settings-current-focus-section.tsx` — No issues found.
- `settings-profile-header.tsx` — No issues found.
- `settings-voice-section.tsx` — No issues found.
- `settings-account.tsx` — No issues found.
- `settings-billing-content.tsx` — Minor: `transactions: ActionTransaction[] = billingSummary.recentTransactions` is a redundant reassignment used for inline type annotation; the cast is safe and the pattern is acceptable. The `tierOrder: SubscriptionTier[]` array cast on a known literal array is also safe.
- `settings-billing.tsx` — Thin wrapper; no issues.
