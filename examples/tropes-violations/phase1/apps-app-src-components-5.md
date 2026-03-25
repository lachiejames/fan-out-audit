# Tropes Audit: apps/app/src/components (batch 5)

Files audited:

- `apps/app/src/components/core-profile-key-people-card.tsx`
- `apps/app/src/components/core-profile-preferences-card.tsx`
- `apps/app/src/components/error-boundary.tsx`
- `apps/app/src/components/feature-error-boundary.tsx`
- `apps/app/src/components/focus-mode.tsx`
- `apps/app/src/components/full-page-error-fallback.tsx`
- `apps/app/src/components/inbox-empty-state.tsx`
- `apps/app/src/components/inline-error-fallback.tsx`

---

## Findings

### 1. "surface" as a verb

**File**: `apps/app/src/components/inbox-empty-state.tsx`, line 68

**Verbatim text**:

> SlopWeaver will surface action items as it learns which messages need follow-up.

**Trope**: "Surface" used as a verb ("surface action items") is a widespread AI/product-management cliche. It adds no meaning over plainer alternatives.

**Suggested fix**: "SlopWeaver will flag action items as it learns which messages need a reply."

---

### 2. Vague "learning your patterns" copy (two instances)

**File**: `apps/app/src/components/inbox-empty-state.tsx`

**Instance A** (line 106, `no_integrations` state):

> Connect your tools so SlopWeaver can start learning your communication patterns.

**Instance B** (line 126, default `no_messages` state):

> Connect your tools so SlopWeaver can start learning your patterns.

**Trope**: Both phrases use the passive "start learning your [X] patterns" construction that is pervasive in AI product copy. "Communication patterns" and "your patterns" are vague and say nothing concrete about what the product actually does. The two strings are also near-duplicates of each other, suggesting copy was written once and copied without revision.

**Suggested fix for Instance A**: "Connect your tools so SlopWeaver can start triaging messages and drafting replies in your voice."

**Suggested fix for Instance B**: "Connect your tools so SlopWeaver can start handling the busywork." (or align with Instance A's fix)

---

## Files with no findings

- `core-profile-key-people-card.tsx` - No user-facing prose; all text is data-driven labels ("Key People", count strings, relationship and platform badges).
- `core-profile-preferences-card.tsx` - Labels only ("Response Length", "Default Formality", "Emoji Usage", "Quick Replies", "Enabled"/"Disabled", "Balanced"/"Concise"/"Detailed"). No prose.
- `error-boundary.tsx` - No user-facing text rendered directly; delegates to child fallback components.
- `feature-error-boundary.tsx` - No user-facing text rendered directly; thin wrapper around `ErrorBoundary`.
- `focus-mode.tsx` - User-facing strings ("Session complete", "You focused for {N} minutes", "Take 5 min break", "Done", "Focusing on", "Remaining", "Duration", "Focus On", "No pending tasks", "End Focus") are all direct and functional. No tropes.
- `full-page-error-fallback.tsx` - "Something went wrong" and "The app failed to load this screen. No messages were sent. Refresh the page or go back to the inbox." are clear and accurate. The branding tagline ("The AI that proves it's learning you") appears as a footer but is the product tagline, not incidental copy.
- `inline-error-fallback.tsx` - "Failed to load this section" and "Retry" are minimal and correct. No tropes.
