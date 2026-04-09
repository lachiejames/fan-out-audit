# Type-Safety Audit: apps-app-src-components-26

**Files audited:**

- `apps/app/src/components/voice/message-tts.types.ts`
- `apps/app/src/components/voice/message-tts.utils.ts`
- `apps/app/src/components/voice/use-message-tts.ts`
- `apps/app/src/components/voice/use-voice-conversation.ts`
- `apps/app/src/components/voice/use-voice-vocabulary.ts`
- `apps/app/src/components/voice/voice-control-policy.ts`
- `apps/app/src/components/voice/voice-conversation-composer.tsx`
- `apps/app/src/components/voice/voice-conversation-debug.ts`

---

## Findings

### 1. `(details as { code?: unknown }).code` cast pattern — `message-tts.utils.ts` line 215–216

**Category:** Avoidable type casts (`as SomeType`)

**Location:** `apps/app/src/components/voice/message-tts.utils.ts:215`

**Description:**
The pattern `(details as { code?: unknown }).code` casts an `unknown` value to access a property. While reaching for `unknown` is correct, the cast to an intermediate object shape is redundant — a type guard would be safer and more readable.

**Suggested fix:**

```typescript
function hasCode(v: unknown): v is { code: unknown } {
  return typeof v === "object" && v !== null && "code" in v;
}
const code = hasCode(details) ? details.code : undefined;
```

---

### 2. Untyped global augmentation for `__SLOPWEAVER_VOICE_METRICS__` — `message-tts.utils.ts` line 429–432

**Category:** Unsafe `any` / `unknown` usage

**Location:** `apps/app/src/components/voice/message-tts.utils.ts:429`

**Description:**
`window.__SLOPWEAVER_VOICE_METRICS__` is assigned via an untyped property access. A global type augmentation in a `.d.ts` file would make this type-safe and IDE-discoverable.

**Suggested fix:**
In a global type declaration file (e.g., `apps/app/src/types/globals.d.ts`):

```typescript
interface VoiceMetrics {
  // fields...
}
interface Window {
  __SLOPWEAVER_VOICE_METRICS__?: VoiceMetrics;
}
```

---

### 3. Repeated cast `(timedResult.body as { details?: unknown } | null)?.details` — `use-message-tts.ts` lines 215, 297

**Category:** Avoidable type casts (`as SomeType`)

**Location:** `apps/app/src/components/voice/use-message-tts.ts:215,297`

**Description:**
The same cast pattern for extracting `details` from an error response body appears in multiple places across the voice module (`use-message-tts.ts`, `message-tts.api.ts`, `message-tts.utils.ts`). This is a sign that the ts-rest error response type for non-200 statuses is not strongly typed.

**Suggested fix:**
Extract a single helper function (ideally in `message-tts.utils.ts`):

```typescript
export function extractErrorDetails(body: unknown): unknown {
  if (typeof body === "object" && body !== null && "details" in body) {
    return (body as { details: unknown }).details;
  }
  return undefined;
}
```

Then call `extractErrorDetails(result.body)` at each use site. This consolidates the cast in one validated place.

---

### 4. `onDebug: (...args: unknown[])` rest parameter — `use-voice-conversation.ts` line 207

**Category:** Unsafe `any` / `unknown` usage

**Location:** `apps/app/src/components/voice/use-voice-conversation.ts:207`

**Description:**
`onDebug: (...args: unknown[]) => void` accepts any arguments. If the ElevenLabs SDK types `onDebug` with specific parameter types, using the SDK's type directly would be safer. If the SDK leaves it untyped, `unknown[]` is acceptable but worth a verification note.

**Suggested action:** Verify that the ElevenLabs `@elevenlabs/react` SDK types the `onDebug` callback. If it does, use the SDK type; if it doesn't, `unknown[]` is the correct approach.

---

### 5. (HIGH) Duplicate `DetailedStatus` type declaration — `voice-conversation-composer.tsx` line 15

**Category:** Duplicate type definitions

**Location:** `apps/app/src/components/voice/voice-conversation-composer.tsx:15`

**Description:**
`DetailedStatus` is declared locally in `voice-conversation-composer.tsx` but is already exported from `voice-conversation.types.ts` (line 91). This creates a duplicate type that may drift from the canonical definition.

**Suggested fix:**

```typescript
// Remove local declaration in voice-conversation-composer.tsx
// Replace with:
import type { DetailedStatus } from "@/components/voice/voice-conversation.types";
```

---

### 6. Window cast augmentations for debug globals — `voice-conversation-debug.ts` lines 49–51, 70–72

**Category:** Avoidable type casts (`as SomeType`)

**Location:** `apps/app/src/components/voice/voice-conversation-debug.ts:49,70`

**Description:**
`window as Window & { __SLOPWEAVER_LAST_VOICE_START__?: Record<string, unknown> }` and similar patterns inline the window type augmentation at each assignment. This is repeated across `message-tts.utils.ts` and `voice-conversation-debug.ts`.

**Suggested fix:** Consolidate all `__SLOPWEAVER_*` debug properties into the global `Window` declaration in `apps/app/src/types/globals.d.ts` (or equivalent). Then the assignments require no cast.

---

## Clean Files

- `message-tts.types.ts` — Clean type definitions; no issues.
- `voice-control-policy.ts` — No issues found.
