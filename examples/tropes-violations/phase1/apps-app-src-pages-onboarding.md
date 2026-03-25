# Trope Audit: apps/app/src/pages/onboarding

Files audited:

- `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/onboarding/connect-step.tsx`
- `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/onboarding/page.tsx`
- `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/onboarding/step-header.tsx`
- `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/onboarding/step-indicator.tsx`
- `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/onboarding/welcome-step.tsx`

---

## page.tsx

No user-facing strings in this file. It is a container component handling step routing, effects, and analytics tracking.

**Findings:** None.

---

## step-header.tsx

Renders the logo and step label. Step label text comes from `STEP_META` in `onboarding.constants` (not in scope). The only user-facing text surfaced here is whatever `STEP_META[currentStep].label` resolves to, which is outside these files.

**Findings:** None in this file.

---

## step-indicator.tsx

### User-facing strings

| Location        | Text                           |
| --------------- | ------------------------------ |
| aria-label      | "Step {n} of {total}: {label}" |
| Visible counter | "{n} of {total}"               |

Functional aria and display text only. No tropes.

**Findings:** None.

---

## connect-step.tsx

### User-facing strings inventory

| Location                  | Text                                                             |
| ------------------------- | ---------------------------------------------------------------- |
| Section heading           | "Connect a tool to start learning"                               |
| Section subtext           | "Each tool you connect gives SlopWeaver more data to work with." |
| Back button               | "Back"                                                           |
| Continue button (desktop) | "Continue to Inbox"                                              |
| Continue button (mobile)  | "Continue"                                                       |
| Skip button (desktop)     | "Skip — connect later in Settings"                               |
| Skip button (mobile)      | "Skip"                                                           |

### Analysis

- **"Connect a tool to start learning"**: Direct and functional. "Learning" is accurate product language (the AI learns from connected data), not marketing fluff.
- **"Each tool you connect gives SlopWeaver more data to work with."**: Honest and concrete, explains the mechanism. No trope.

**Findings:** None.

---

## welcome-step.tsx

### User-facing strings inventory

| Location       | Text                                                                                                                                                                |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Headline       | "Set up your AI"                                                                                                                                                    |
| Body paragraph | "SlopWeaver will learn how you work. Over the next few days, you'll see your acceptance rate start to climb. OAuth only, read-only by default. Disconnect anytime." |
| Primary CTA    | "Connect your first tool"                                                                                                                                           |
| Skip link      | "Skip — connect later in Settings"                                                                                                                                  |

### Analysis

This is the highest-risk surface in the onboarding flow. Reviewing each string:

- **"Set up your AI"**: Clean, imperative, direct. No trope.
- **"SlopWeaver will learn how you work."**: Accurate product claim. Not hype.
- **"Over the next few days, you'll see your acceptance rate start to climb."**: Concrete, measurable, time-bound. Excellent - this is exactly the kind of claim that replaces generic AI promises.
- **"OAuth only, read-only by default. Disconnect anytime."**: Transparent, concrete, trust-building. No trope.
- **"Connect your first tool"**: Direct CTA. No trope.
- **"Skip — connect later in Settings"**: Functional. No trope.

The code comment on line 14 reads `{/* Concrete constraints - not marketing fluff */}` confirming intentional awareness of the anti-trope standard.

**Findings:** None. This is a strong example of on-brand, trope-free copy.
