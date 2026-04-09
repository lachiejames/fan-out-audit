# Type-Safety Audit: apps-app-src-components-28

**Files audited:**

- `apps/app/src/components/voice/voice-live-rows.ts`
- `apps/app/src/components/voice/voice-privacy-consent.utils.ts`
- `apps/app/src/components/voice/voice-settings.schema.ts`

---

## Findings

### 1. Re-export of `isPlaceholderVoiceTranscript` — `voice-live-rows.ts` line 13

**Category:** Duplicate type definitions / re-export anti-pattern

**Location:** `apps/app/src/components/voice/voice-live-rows.ts:13`

**Description:**
`export { isPlaceholderVoiceTranscript }` is a backward-compatibility re-export from `voice-live-row-text`. The file comment acknowledges this is "for existing consumers." Per `docs/agent-rules/code-organization.md`, backward-compatibility re-exports are banned — all consumers should import directly from the source file.

**Suggested fix:** Remove the re-export and update all import sites to import `isPlaceholderVoiceTranscript` directly from `voice-live-row-text` (or its canonical source).

---

### 2. `v as VoiceProfileId` inside a type predicate refinement — `voice-settings.schema.ts` line 23

**Category:** Avoidable type casts (`as SomeType`)

**Location:** `apps/app/src/components/voice/voice-settings.schema.ts:23`

**Description:**

```typescript
.refine((v): v is VoiceProfileId => VOICE_PROFILE_ID_LIST.includes(v as VoiceProfileId))
```

The `v as VoiceProfileId` inside the `.includes()` call is required by TypeScript's TS1230 limitation: type predicates cannot destructure, so the `v` is the raw validated string. The `as VoiceProfileId` cast is needed to satisfy `.includes()` which expects the array's element type.

**Assessment:** This pattern is a legitimate TS1230 workaround and is explicitly permitted by the project's TypeScript rules. No change required. Document the reason with a comment if one is not already present.

---

## Clean Files

- `voice-privacy-consent.utils.ts` — No issues found.

---

## Summary Note on Voice Module

Across slices 25–28 (the voice component group), there is a recurring theme of repeated `(body as { details?: unknown })` and `(value as { message?: unknown })` casts for extracting fields from non-200 ts-rest response bodies. These casts exist in at least five separate files:

- `message-tts.api.ts`
- `message-tts.utils.ts`
- `use-message-tts.ts` (two occurrences)
- `voice-conversation.api.ts`

The highest-value improvement across the voice module would be to create a single shared `extractVoiceApiError` utility function in `message-tts.utils.ts` that centralizes this cast with a proper type guard, eliminating the repeated unsafe patterns at each call site.
