# Type-Safety Audit: apps-app-src-components-25

**Files audited:**

- `apps/app/src/components/triage/IdleWarningBanner.tsx`
- `apps/app/src/components/triage/triage-action.utils.ts`
- `apps/app/src/components/triage/triage-learning-summary.utils.ts`
- `apps/app/src/components/triage/triage-stat-card.tsx`
- `apps/app/src/components/triage/TriageTriggerBanner.tsx`
- `apps/app/src/components/triage/TriageTriggerProvider.tsx`
- `apps/app/src/components/voice/message-tts.api.ts`
- `apps/app/src/components/voice/message-tts.stream.ts`

---

## Findings

### 1. (HIGH) `Object.keys(availability) as TriageAction[]` — `triage-action.utils.ts` line 38

**Category:** Avoidable type casts (`as SomeType`)

**Location:** `apps/app/src/components/triage/triage-action.utils.ts:38`

**Description:**
`Object.keys()` always returns `string[]`. Casting the result to `TriageAction[]` will silently accept any object with extra or wrong keys at runtime. If `availability` is typed as `Record<TriageAction, boolean>`, a safer approach uses typed key iteration:

**Suggested fix:**

```typescript
import type { TriageAction } from "@slopweaver/contracts";

// Option A: type predicate guard
const isTriageAction = (key: string): key is TriageAction => TRIAGE_ACTIONS.includes(key as TriageAction);

const actions = Object.keys(availability).filter(isTriageAction);

// Option B: if availability is already Record<TriageAction, boolean>, cast the param type:
function getAvailableActions({ availability }: { availability: Record<TriageAction, boolean> }): TriageAction[] {
  return (Object.keys(availability) as TriageAction[]).filter((key) => availability[key]);
}
// (Option B is safe only when the object is structurally guaranteed to have only TriageAction keys)
```

---

### 2. `Record<string, string>` for `STATUS_TO_ACTION` with known key domain — `triage-learning-summary.utils.ts` line 11

**Category:** `Record<string, ...>` where stricter types would be safer

**Location:** `apps/app/src/components/triage/triage-learning-summary.utils.ts:11`

**Description:**
`STATUS_TO_ACTION: Record<string, string>` maps triage status strings to action strings. Both the keys and values are drawn from known contract unions (`TriageStatus` → `TriageAction`). Using `Record<string, string>` loses exhaustiveness; adding a new status without a mapping will not cause a compile error.

**Suggested fix:**

```typescript
import type { TriageAction, TriageStatus } from "@slopweaver/contracts";

const STATUS_TO_ACTION: Record<TriageStatus, TriageAction> = {
  archived: "archive",
  // ...
};
```

---

### 3. Missing explicit return type on exported `StatCard` — `triage-stat-card.tsx`

**Category:** Unsafe `unknown` / `any` usage (implicit return type)

**Location:** `apps/app/src/components/triage/triage-stat-card.tsx`

**Description:**
The `StatCard` component is exported via `memo(...)` without an explicit return type annotation. Per the project rule (explicit return types on all exported functions), the inner function passed to `memo` should declare `JSX.Element` (or `React.ReactElement`) as its return type.

**Suggested fix:**

```typescript
export const StatCard = memo(function StatCard({ ... }: Props): JSX.Element {
  return <div>...</div>;
});
```

---

### 4. Double-cast pattern on `response.json()` — `message-tts.api.ts` line 62

**Category:** Avoidable type casts (`as SomeType`)

**Location:** `apps/app/src/components/voice/message-tts.api.ts:62`

**Description:**
`(await response.json()) as unknown` followed by `(body as { message?: unknown }).message` performs two casts. The first cast to `unknown` is intentional and safe, but the subsequent property access cast introduces potential unsafety. A type guard would be cleaner:

**Suggested fix:**

```typescript
function hasMessage(v: unknown): v is { message: unknown } {
  return typeof v === "object" && v !== null && "message" in v;
}

const body: unknown = await response.json();
const message = hasMessage(body) ? body.message : undefined;
```

---

### 5. `c.buffer as ArrayBuffer` on deprecated `buffer` property — `message-tts.stream.ts` line 77

**Category:** Avoidable type casts (`as SomeType`)

**Location:** `apps/app/src/components/voice/message-tts.stream.ts:77`

**Description:**
The `buffer` property on an `AudioBuffer` instance is a deprecated API. The cast `c.buffer as ArrayBuffer` is technically correct but documents that a deprecated API is in use. Consider adding a comment explaining why `buffer` is used instead of a modern alternative, or migrate to a non-deprecated API.

**Suggested action:** No type-safety change required, but document the deprecation rationale in a code comment.

---

## Clean Files

- `IdleWarningBanner.tsx` — No issues found.
- `TriageTriggerBanner.tsx` — No issues found.
- `TriageTriggerProvider.tsx` — No issues found.
