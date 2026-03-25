# Trope Audit: apps/app/src/components (Slice 66)

Files audited:

- offline-auth-fallback.tsx
- page-loader.tsx
- require-auth.tsx
- require-onboarded.tsx
- root-redirect.tsx
- settings-profile.tsx
- sidebar.tsx
- startup-recovery-fallback.tsx

---

## offline-auth-fallback.tsx

No findings. Copy is direct and functional ("Can't reach SlopWeaver", "We couldn't reach the API during startup. Retry the connection, or go to login if you want to continue.", "Retry connection", "Go to login").

---

## page-loader.tsx

No findings. "Loading your workspace..." is plain status copy.

---

## require-auth.tsx

No user-facing JSX text. Renders route guards only; all visible text is delegated to child components.

---

## require-onboarded.tsx

No user-facing JSX text. Pure route guard with no rendered copy.

---

## root-redirect.tsx

No user-facing JSX text. Pure redirect logic with no rendered copy.

---

## settings-profile.tsx

### Violation 1

**File**: `settings-profile.tsx`
**Line**: 96
**Text**: `"Your AI profile is still being built. SlopWeaver needs more activity to learn your patterns."`
**Trope**: "still being built" + "learn your patterns" -- classic AI product hedging language. The phrase "needs more activity to learn" is a stock AI onboarding trope that sounds generic and self-referential.
**Suggested fix**: "Your profile isn't ready yet. Connect more tools and SlopWeaver will have enough to go on." or simply "Not enough activity yet to build your profile."

---

## sidebar.tsx

No findings. Nav labels ("Today", "Inbox", "Triage", "Queue", "Tasks", "Assistant", "Analytics", "Settings", "Search...") are all direct UI labels. No AI tropes present.

---

## startup-recovery-fallback.tsx

No findings. Copy is functional and direct ("Startup is taking longer than expected", "SlopWeaver is still loading your workspace. You can retry, go to login, or copy diagnostics for support.", "Retry", "Copy diagnostics", "Go to login", "Copying diagnostics...", "Preparing diagnostics...", "Diagnostics copied", "Failed to copy diagnostics", "Clipboard unavailable; diagnostics saved instead"). No inflated AI language.
