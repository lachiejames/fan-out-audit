# Tropes Audit: apps/app/src/components/auth (slice 1)

Files audited:

- `apps/app/src/components/auth/auth-branding-sidebar.tsx`
- `apps/app/src/components/auth/auth-divider.tsx`
- `apps/app/src/components/auth/auth-error-banner.tsx`
- `apps/app/src/components/auth/auth-input.tsx`
- `apps/app/src/components/auth/auth-loading-skeleton.tsx`
- `apps/app/src/components/auth/auth-logo.tsx`
- `apps/app/src/components/auth/auth-page-layout.tsx`
- `apps/app/src/components/auth/google-button.tsx`

---

## auth-branding-sidebar.tsx

### Finding 1

**File**: `apps/app/src/components/auth/auth-branding-sidebar.tsx`
**Line**: 14 (featureSets.login[0])
**Text**: `"Watch your AI improve over time"`
**Trope**: Vague AI agency. "Your AI improve" is unspecific and anthropomorphizes the system. Does not say what improves or how.
**Suggested fix**: `"Track acceptance rate and accuracy as the system learns your style"`

### Finding 2

**File**: `apps/app/src/components/auth/auth-branding-sidebar.tsx`
**Line**: 23 (featureSets.signup[0])
**Text**: `"Your AI starts learning from messages you sync"`
**Trope**: "AI starts learning" is vague and mystifying. Does not describe the actual mechanism (reading patterns from accepted/edited drafts).
**Suggested fix**: `"SlopWeaver reads your synced messages to match your writing style"`

### Finding 3

**File**: `apps/app/src/components/auth/auth-branding-sidebar.tsx`
**Line**: 36 (subtitles.signup)
**Text**: `"Connect your tools. Watch the AI learn."`
**Trope**: "Watch the AI learn" frames passive data accumulation as a spectacle. Vague and creates inflated expectations.
**Suggested fix**: `"Connect your tools. See accuracy improve with every interaction."`

---

## auth-divider.tsx

No findings.

---

## auth-error-banner.tsx

No findings.

---

## auth-input.tsx

No findings.

---

## auth-loading-skeleton.tsx

No findings.

---

## auth-logo.tsx

No findings.

---

## auth-page-layout.tsx

No findings.

---

## google-button.tsx

No findings.
