# Trope Audit: apps/app/src/components/ai-chat-widget (Slice 68)

Files audited:

- model-selector.tsx
- voice-settings-content.tsx
- voice-settings-menu.tsx

---

## model-selector.tsx

### Violation 1

**File**: `model-selector.tsx`
**Line**: 120
**Text**: `"Supports extended reasoning"`
**Trope**: "Extended reasoning" is AI-industry jargon. Users don't know or care what "extended reasoning" means as a technical capability label; it reads as marketing speak lifted from model provider documentation.
**Suggested fix**: Describe the practical benefit instead, e.g. "Better at complex, multi-step problems" or drop the capability line entirely and keep only the tier-restriction note.

### Violation 2

**File**: `model-selector.tsx`
**Line**: 120
**Text**: `"No extended reasoning"`
**Trope**: Negative form of the same AI jargon. Doubly unhelpful -- it tells users about the absence of something they may not understand.
**Suggested fix**: Omit entirely, or use a plain comparative like "Faster, lighter model".

---

## voice-settings-content.tsx

### Violation 1

**File**: `voice-settings-content.tsx`
**Lines**: 250-253
**Text**: `"Generating assistant voice"` / `"Playing assistant voice"` / `"Assistant voice ready"`
**Trope**: "Assistant voice" is a redundant qualifier in this context -- the user is already in a voice settings panel. The phrasing also uses "assistant" as a noun modifier, which is a common AI product verbal tic.
**Suggested fix**: "Generating audio..." / "Playing..." / "Ready to play" -- shorter and direct.

---

## voice-settings-menu.tsx

### Violation 1

**File**: `voice-settings-menu.tsx`
**Line**: 182
**Text**: `"Input, output, recording mode, and v3 voice profile preview."`
**Trope**: "v3 voice profile preview" is internal versioning surfaced as user-facing copy. Users have no context for what "v3" means and it reads like a developer placeholder that was never cleaned up.
**Suggested fix**: "Configure voice input, output, and recording mode. Preview voice profiles before selecting."

### Violation 2

**File**: `voice-settings-menu.tsx`
**Line**: 109
**Text**: `"Voice provider is unavailable right now."` (repeated as profile `unavailableReason`)
**Trope**: "Voice provider" is infrastructure terminology leaked into user-facing copy. Users see a feature, not a provider.
**Suggested fix**: "Voice is unavailable right now." or "Voice not available -- try again shortly."

### Violation 3

**File**: `voice-settings-menu.tsx`
**Lines**: 114, 120, 123, 132, 133
**Text**: `"Voice provider is unavailable right now. Retry in a moment."` / `"Unable to load curated voices right now."` / `"Voice catalog is unavailable for this account."` / `"Voice catalog failed"` (toast title)
**Trope**: "Voice catalog" and "voice provider" are internal data-model terms. "Curated voices" is unnecessary filler adjective. Toast titles with "failed" as a standalone word are abrupt.
**Suggested fix**: "Couldn't load voices. Try again." for the error state; "Voice unavailable" for the toast title; "Voices aren't available for your account." for the 402/429 case.
