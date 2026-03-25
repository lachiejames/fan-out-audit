# Tropes Audit: apps/app/src/components (batch 4)

Files audited:

- `ai-tool-row.utils.tsx`
- `ai-typing-indicator.tsx`
- `chunk-error-fallback.tsx`
- `connect-prompt-banner.tsx`
- `connection-status-banner.tsx`
- `core-profile-communication-card.tsx`
- `core-profile-display.tsx`
- `core-profile-identity-card.tsx`

---

## Findings

### 1. `ai-typing-indicator.tsx` — line 49

**Text**: `"Thinking..."`

**Trope**: AI anthropomorphization. Attributing human cognitive process ("thinking") to the system is a cliche used across nearly every AI product. It implies internal experience the system does not have and has become noise users ignore.

**Suggested fix**: Replace with a concrete status that describes what is actually happening, e.g. `"Working on it..."`, `"Generating..."`, or simply omit the label and rely on the animated dots/progress bar alone. If the `status` prop is populated, it will already show below this label, making the generic fallback doubly redundant.

---

## No further findings

The remaining files contain no AI writing tropes in user-facing text:

- `ai-tool-row.utils.tsx` — status strings ("running", "needs approval", "approved", "canceled", "failed", "complete") are functional labels with no trope language.
- `chunk-error-fallback.tsx` — error copy is plain and direct. The tagline on line 40 (`"SlopWeaver - The AI that proves it's learning you"`) is a brand tagline, not a UI copy trope.
- `connect-prompt-banner.tsx` — "Connect your tools so SlopWeaver can start learning your patterns." is clear product language. "learning your patterns" is on-brand and factual for this product.
- `connection-status-banner.tsx` — "Reconnecting to server..." and "Unable to connect. Your changes may not be saved." are direct, accurate status messages.
- `core-profile-communication-card.tsx` — all labels are data field names ("Style", "Formality Level", "Greeting", "Closing"). No trope language.
- `core-profile-display.tsx` — layout-only component, no user-facing text.
- `core-profile-identity-card.tsx` — all labels are data field names ("Team:", "Timezone:", "Hours:"). No trope language.
