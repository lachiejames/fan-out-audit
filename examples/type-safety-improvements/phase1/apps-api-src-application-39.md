# Audit: apps-api-src-application-39

**Files inspected**: 8
**Findings**: 5

## Summary

The user-service and onboarding-service files are clean. The voice errors module has a structural duplication issue — shared error variant shapes are copy-pasted between two separate union types rather than extracted to a shared base. The voice-catalog service defines two local types that duplicate or partially overlap with already-exported contracts types. The voice-conversation-session service inlines the billing-state literal union three times instead of reusing the `VoiceBillingState` type exported from the adjacent billing utils file. The voice-observability util's return type is `Record<string, unknown>` where a stricter typed shape would be both safe and more useful to callers.

---

## Findings

### Finding 1: Inline billing-state literal union duplicated three times instead of reusing `VoiceBillingState`

- **File**: `apps/api/src/application/voice/services/voice-conversation-session.service.ts:42`, `:97`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `VoiceBillingState = "ok" | "low_balance" | "text_only"` is exported from `voice-conversation-billing.utils.ts` (line 13), but `VoiceConversationSession.billingState` (line 42) and the `createSession` param object (line 97) each repeat the inline literal union. If a new state is ever added (e.g. `"suspended"`), all three sites must be updated in sync.
- **Suggestion**: Import `VoiceBillingState` from `./voice-conversation-billing.utils` and use it in both the interface field and the parameter type.
- **Evidence**:

```typescript
// voice-conversation-session.service.ts line 42
billingState?: "ok" | "low_balance" | "text_only" | undefined;

// voice-conversation-session.service.ts line 97
billingState?: "ok" | "low_balance" | "text_only";

// voice-conversation-billing.utils.ts line 13 — already exported
export type VoiceBillingState = "ok" | "low_balance" | "text_only";
```

---

### Finding 2: `VoiceCatalogProfileEntry` and `VoiceCatalogPayload` duplicate contracts types

- **File**: `apps/api/src/application/voice/services/voice-catalog.service.ts:16-29`
- **Category**: sdk-type-duplication
- **Impact**: medium
- **Description**: `VoiceCatalogProfileEntry` is a hand-written interface with fields `available`, `description`, `id`, `label`, `previewText`, `unavailableReason`, `voiceId`. `@slopweaver/contracts` already exports `VoiceCatalogResponse` (inferred from `voiceCatalogResponseSchema`) whose `profiles` array element schema (`voiceCatalogProfileSchema`) is the canonical definition of these same fields. `VoiceCatalogPayload` with `{ modelId: string; profiles: VoiceCatalogProfileEntry[] }` duplicates `VoiceCatalogResponse`. If the contract schema evolves (e.g. a new field is added), the local types silently diverge.
- **Suggestion**: Replace both local types with the imported contracts type: `import type { VoiceCatalogResponse } from "@slopweaver/contracts"` and change the return type of `getCatalog` to `Promise<VoiceCatalogResponse>`.
- **Evidence**:

```typescript
// voice-catalog.service.ts — local hand-written types
type VoiceCatalogProfileEntry = {
  available: boolean;
  description: string;
  id: VoiceProfileId;
  label: string;
  previewText: string;
  unavailableReason?: string | undefined;
  voiceId?: string | undefined;
};

export type VoiceCatalogPayload = {
  modelId: string;
  profiles: VoiceCatalogProfileEntry[];
};

// packages/contracts/src/contracts/voice/schemas.ts line 56-65 — already canonical
export const voiceCatalogProfileSchema = voiceProfileSchema.extend({
  available: z.boolean(),
  unavailableReason: z.string().min(1).optional(),
  voiceId: z.string().min(1).optional(),
});
export const voiceCatalogResponseSchema = z.object({
  modelId: z.string().min(1),
  profiles: z.array(voiceCatalogProfileSchema).min(1),
});
export type VoiceCatalogResponse = z.infer<typeof voiceCatalogResponseSchema>;
```

---

### Finding 3: `ResolvedProfiles` uses `Record<VoiceProfileId, …>` with an anonymous inline value shape

- **File**: `apps/api/src/application/voice/services/voice-catalog.service.ts:7-14`
- **Category**: record-weakening
- **Impact**: low
- **Description**: The `ResolvedProfiles` type's value shape `{ available: boolean; unavailableReason?: string; voiceId?: string }` is a partial subset of `VoiceCatalogProfileEntry` (defined 9 lines below in the same file). Because `Record` flattens this to an inline anonymous object, callers that read an entry get no named type, making it harder to understand the relationship between resolved profiles and catalog entries. Additionally, using `{} as ResolvedProfiles` to seed the `reduce` accumulator (line 49) is an unsafe cast — TypeScript accepts it, but the object is genuinely empty before population.
- **Suggestion**: Extract the value shape as a named type (e.g. `type ResolvedVoiceProfile`) and seed the reducer with `Object.fromEntries(...)` or type the accumulator differently to avoid the cast.
- **Evidence**:

```typescript
type ResolvedProfiles = Record<
  VoiceProfileId,
  {
    available: boolean;
    unavailableReason?: string | undefined;
    voiceId?: string | undefined;
  }
>;

// line 49 — unsafe seed cast
}, {} as ResolvedProfiles);
```

---

### Finding 4: Duplicate provider-error variant shapes across `VoiceTranscriptionError` and `VoiceSynthesisError`

- **File**: `apps/api/src/application/voice/errors/voice.errors.ts:1-17`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: Five error variant shapes — `VOICE_PROVIDER_QUOTA_EXCEEDED`, `VOICE_PROVIDER_RATE_LIMITED`, `VOICE_PROVIDER_UNAVAILABLE`, `VOICE_PROVIDER_UNAUTHORIZED`, `VOICE_UPSTREAM_FAILURE` — are copy-pasted verbatim in both `VoiceTranscriptionError` and `VoiceSynthesisError`. This means every structural change (e.g. adding a `retryAfter` field to `VOICE_PROVIDER_RATE_LIMITED`) must be made in two places, and the `VoiceErrors` factory object has ten functions (five pairs) instead of five. The `VoiceErrors` factory also positionally encodes which error type to return via `Synthesis`/`Transcription` suffix naming, which is fragile.
- **Suggestion**: Extract the shared variants into a `VoiceProviderError` union, then compose: `type VoiceTranscriptionError = { code: "VOICE_AUDIO_INVALID"; ... } | { code: "VOICE_TRANSCRIPTION_EMPTY"; ... } | VoiceProviderError` and similarly for synthesis. Factory functions for the shared variants can be defined once.
- **Evidence**:

```typescript
// Repeated verbatim in both union types:
| { code: "VOICE_PROVIDER_QUOTA_EXCEEDED"; message: string; status?: number }
| { code: "VOICE_PROVIDER_RATE_LIMITED"; message: string; status?: number }
| { code: "VOICE_PROVIDER_UNAVAILABLE"; message: string; status?: number }
| { code: "VOICE_PROVIDER_UNAUTHORIZED"; message: string; status?: number }
| { code: "VOICE_UPSTREAM_FAILURE"; message: string; status?: number }
```

---

### Finding 5: `shapeVoiceObservabilityEvent` return type `Record<string, unknown>` loses all field knowledge

- **File**: `apps/api/src/application/voice/services/voice-observability.utils.ts:39`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The function manually populates a `Record<string, unknown>` from a strongly-typed `VoiceObservabilityEvent` input. Because the return type is `Record<string, unknown>`, callers lose the type information they already had on the input. All accesses on the returned object require `as` casts or `unknown` checks. Since the function's purpose is to strip `undefined` fields (not change types), the return type could be `Partial<Omit<VoiceObservabilityEvent, "operation">> & { msg: string; operation: VoiceObservabilityEvent["operation"] }` or, simpler, the same `VoiceObservabilityEvent & { msg: string }` shape with optional fields. The logger likely accepts `Record<string, unknown>` so this is a boundary concern, but the return type overly erases structure.
- **Suggestion**: Tighten the return type to `{ msg: string } & Partial<VoiceObservabilityEvent>` (which is a subtype of `Record<string, unknown>` and satisfies logger interfaces), preserving field awareness at call sites.
- **Evidence**:

```typescript
export function shapeVoiceObservabilityEvent({ event }: { event: VoiceObservabilityEvent }): Record<string, unknown> {
  const result: Record<string, unknown> = {
    msg: `voice:${event.operation}`,
    operation: event.operation,
  };
  // ... manual conditional field population
  return result;
}
```
