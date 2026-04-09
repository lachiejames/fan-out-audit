# Type-Safety Audit: apps-app-src-components-27

**Files audited:**

- `apps/app/src/components/voice/voice-conversation-failure.utils.ts`
- `apps/app/src/components/voice/voice-conversation-handlers.utils.ts`
- `apps/app/src/components/voice/voice-conversation-provider.tsx`
- `apps/app/src/components/voice/voice-conversation.api.ts`
- `apps/app/src/components/voice/voice-conversation.types.ts`
- `apps/app/src/components/voice/voice-input-recorder.ts`
- `apps/app/src/components/voice/voice-input.api.ts`
- `apps/app/src/components/voice/voice-input.utils.ts`

---

## Findings

### 1. `context as Record<string, unknown>` on `unknown` input — `voice-conversation-failure.utils.ts` line 32

**Category:** Avoidable type casts (`as SomeType`)

**Location:** `apps/app/src/components/voice/voice-conversation-failure.utils.ts:32`

**Description:**
`context as Record<string, unknown>` casts an `unknown` input directly to a `Record` type. This silently succeeds even if `context` is a string, number, or array.

**Suggested fix:**

```typescript
function isRecord(v: unknown): v is Record<string, unknown> {
  return typeof v === "object" && v !== null && !Array.isArray(v);
}

const ctx = isRecord(context) ? context : {};
```

---

### 2. Double string cast on `unknown` close reason — `voice-conversation-handlers.utils.ts` line 100

**Category:** Avoidable type casts (`as SomeType`)

**Location:** `apps/app/src/components/voice/voice-conversation-handlers.utils.ts:100`

**Description:**
`(errorMessage as string) ?? (closeReason as string)` casts two `unknown` values to `string`. If either is `null`, `undefined`, or a non-string, the cast will not throw but will produce a value that silently fails string operations.

**Suggested fix:**

```typescript
const message =
  (typeof errorMessage === "string" ? errorMessage : null) ??
  (typeof closeReason === "string" ? closeReason : null) ??
  "Unknown error";
```

---

### 3. Zustand store values cast to contract types on read — `voice-conversation-provider.tsx` lines 55–56

**Category:** Avoidable type casts (`as SomeType`) / Custom types weaker than contract types

**Location:** `apps/app/src/components/voice/voice-conversation-provider.tsx:55,56`

**Description:**
`voiceConversationTurnMode as VoiceTurnMode` and `voiceProfileId as VoiceProfileId` indicate the Zustand store persists these fields as `string` rather than their contract-typed equivalents. The cast is required at every read site.

**Root cause:** The Zustand store definition likely types `voiceConversationTurnMode` and `voiceProfileId` as `string | null` or `string` instead of `VoiceTurnMode | null` and `VoiceProfileId | null`.

**Suggested fix:** Update the Zustand store type (likely in `apps/app/src/lib/*-store.ts`) to use the contract types directly:

```typescript
import type { VoiceProfileId, VoiceTurnMode } from "@slopweaver/contracts";

interface VoiceStoreState {
  voiceConversationTurnMode: VoiceTurnMode | null;
  voiceProfileId: VoiceProfileId | null;
}
```

Then the casts in `voice-conversation-provider.tsx` become unnecessary.

---

### 4. Error response body cast in `voice-conversation.api.ts` line 54

**Category:** Avoidable type casts (`as SomeType`)

**Location:** `apps/app/src/components/voice/voice-conversation.api.ts:54`

**Description:**
`response.body as { message?: string | string[]; error?: string; details?: unknown }` is cast because ts-rest does not strongly type non-200 response bodies for this endpoint. This is a recurring pattern across the voice module.

**Note:** This is an inherent limitation of ts-rest's error response typing (non-contract status codes are `unknown`). The cast is acceptable but should be extracted to a shared `extractVoiceErrorBody` helper (see related finding in slice 26) to avoid repetition.

---

### 5. `globalThis.crypto as undefined | { randomUUID?: () => string }` — `voice-input.api.ts` line 19

**Category:** Avoidable type casts (`as SomeType`)

**Location:** `apps/app/src/components/voice/voice-input.api.ts:19`

**Description:**
This cast handles the case where `globalThis.crypto` might be unavailable or lack `randomUUID` in older environments. Since the project targets Node.js 22 and modern browsers, `globalThis.crypto` and `crypto.randomUUID()` are always available. This defensive cast may be unnecessary.

**Suggested action:** Verify if the cast is still needed for the target environments (e.g., iOS 15 WebView in Tauri). If modern environments are guaranteed, remove the defensive cast. If not, document the targeted environment in a comment.

---

## Clean Files

- `voice-conversation.types.ts` — This is the canonical type definition file; no issues. It defines `DetailedStatus`, `ConversationStatus`, `VoiceTurnMode`, etc.
- `voice-input-recorder.ts` — Uses deprecated `ScriptProcessorNode` intentionally (with eslint-disable); no type-safety changes needed.
- `voice-input.utils.ts` — No issues found.
