# Audit: apps-api-src-interface-18

**Findings count**: 4
**Summary**: Four issues in the voice conversation layer. `TOOL_STATUS_MAP` uses `Record<string, string>` where a union-keyed type would catch typos. Two object-part extraction functions cast `object` to `Record<string, unknown>` instead of using a proper narrowing utility. The voice handler casts `session.subscriptionStatus` to a full union literal, indicating the session model stores this as a plain `string` rather than the proper contracts union type.

---

## Finding 1

**File**: `apps/api/src/interface/http/voice/voice-conversation-handler.utils.ts`
**Line**: 24
**Category**: Weak `Record<string, …>` type
**Impact**: Medium — keys are a fixed set of known ElevenLabs tool-status strings. Using `Record<string, string>` allows typos in key lookups to compile and silently return `undefined`.

**Description**: `TOOL_STATUS_MAP` maps ElevenLabs tool-call status strings to internal status labels. The set of valid keys is finite and known at compile time. A `Record<string, string>` annotation does not catch misspelled key lookups.

**Suggestion**: Define a union `type ElevenLabsToolStatus = "success" | "error" | ...` for the keys and annotate as `Record<ElevenLabsToolStatus, string>` or use `satisfies`.

**Evidence**:

```typescript
const TOOL_STATUS_MAP: Record<string, string> = {
  success: "completed",
  error: "failed",
  // ...
};
```

---

## Finding 2

**File**: `apps/api/src/interface/http/voice/voice-conversation-message-extraction.utils.ts`
**Line**: 66
**Category**: Type cast (`as`)
**Impact**: Medium — `part` is already narrowed to `object` by the preceding `typeof` check. Casting to `Record<string, unknown>` is a redundant assertion rather than a proper narrowing, and hides that arbitrary property access is not guaranteed.

**Description**: After a `typeof part === "object" && part !== null` check, the code immediately casts `part as Record<string, unknown>` to access properties by string key. A type-guard utility (`isRecord`) would be explicit and reusable.

**Suggestion**: Use a `isRecord(value: unknown): value is Record<string, unknown>` type guard imported from shared utilities, or access properties via `Object.hasOwn` checks.

**Evidence**:

```typescript
const record = part as Record<string, unknown>;
```

---

## Finding 3

**File**: `apps/api/src/interface/http/voice/voice-conversation-message-extraction.utils.ts`
**Line**: 108
**Category**: Type cast (`as`)
**Impact**: Medium — same pattern as Finding 2 applied to `message` objects. Both casts originate from the same root cause: needing to treat narrowed `object` values as property-bag types.

**Description**: `message as Record<string, unknown>` immediately after `typeof message === "object" && message !== null`.

**Suggestion**: Same as Finding 2 — extract and use a shared `isRecord` type guard.

**Evidence**:

```typescript
const record = message as Record<string, unknown>;
```

---

## Finding 4

**File**: `apps/api/src/interface/http/voice/voice-conversation.handler.ts`
**Line**: 709
**Category**: Type cast (`as`)
**Impact**: High — `session.subscriptionStatus` is stored as a plain `string` in the session model rather than the proper union from `@slopweaver/contracts`. The cast to `"active" | "trialing" | "past_due" | "cancelled" | "expired" | "paused"` duplicates the union literal inline and is invisible to refactoring tools.

**Description**: The long inline union cast on `subscriptionStatus` indicates the session DB model or DTO exposes this field as `string`. Any future addition to the `SubscriptionStatus` union in contracts would not be caught by the compiler at this call site.

**Suggestion**: Type `session.subscriptionStatus` as the `SubscriptionStatus` type from `@slopweaver/contracts` (or `shared/types/billing.types`) in the session model/DTO definition, eliminating the inline cast.

**Evidence**:

```typescript
session.subscriptionStatus as "active" | "trialing" | "past_due" | "cancelled" | "expired" | "paused";
```
