# Trope Audit: apps/app/src/components/billing (batch 1)

Files audited:

- action-dashboard.tsx
- action-top-up-modal.tsx
- action-warning-banner-wrapper.tsx
- action-warning-banner.tsx
- ai-transparency-card.tsx
- alert-settings-card.tsx
- alert-settings-skeleton.tsx
- auto-refill-settings.tsx

---

## Findings

### action-warning-banner.tsx

**Stakes inflation / "avoid interruptions" filler**

Line 39 (critical level message):

> "Upgrade to keep working without interruptions."

Line 57 (warning level message):

> "Consider upgrading to avoid interruptions."

"Working without interruptions" is vague stakes inflation. The message is really: you'll run out of actions and AI features will stop. Say that directly. "Avoid interruptions" is a stock SaaS upsell phrase with no specificity.

Suggested rewrites:

- Critical: "Upgrade to keep your action balance topped up."
- Warning: "Consider upgrading before your balance runs out."

**Patronizing / unnecessary title for info level**

Line 51:

> title: "Capacity Update"

The info-level banner is not a capacity update -- it's a usage heads-up. "Capacity Update" reads like a system status noun-stack borrowed from enterprise software. The warning-level title "Usage running low" and critical title "Usage critically low" are plain and correct. The info-level title should match that register.

Suggested rewrite: "Half your usage is gone" or simply "Usage update".

---

### action-top-up-modal.tsx

**"Best Value" badge**

Line 120:

> Best Value

Minor: a badge claiming "Best Value" on a pricing option is a stock e-commerce pattern that signals sales pressure rather than factual comparison. It is not a writing trope per se, but it is the kind of copy pattern the audit was designed to flag. Keeping it is defensible; removing it or replacing with a factual claim (e.g. "Lowest per-action price") would be stronger.

No other tropes found in this file.

---

### action-dashboard.tsx

**"Where your usage went" -- mild pedagogical framing**

Line 239:

> "Where your usage went this period"

This is borderline. It reads like a teacher summarizing a lesson. The heading "Usage Breakdown" already labels the section. The subheading adds nothing and could be cut entirely, or replaced with the period label (e.g. "This billing period").

**"Switch & Save" CTA**

Line 347:

> Switch & Save

Ampersand in a CTA combining two verbs is a compressed sales phrase. Not a trope in the strict sense, but the copy could be cleaner: "Switch to annual" with the savings amount visible in the adjacent text.

---

### auto-refill-settings.tsx

**Redundant enable label description**

Lines 147:

> "Automatically purchase actions when you run low"

This is a near-verbatim repeat of the card subheading on line 138:

> "Automatically purchase actions when balance runs low"

One of them should be cut. The label description under the toggle is redundant noise.

---

### alert-settings-card.tsx

**"assisted actions" noun phrase**

Line 173:

> Current balance: {currentBalance.toLocaleString()} assisted actions

"Assisted actions" is an ornate noun compound. Every other place in these files uses plain "actions". This one instance uses "assisted actions" inconsistently and without reason. Should be "actions".

---

### ai-transparency-card.tsx, action-warning-banner-wrapper.tsx, alert-settings-skeleton.tsx

No findings.

---

## Summary

| File                              | Findings                                    |
| --------------------------------- | ------------------------------------------- |
| action-warning-banner.tsx         | Stakes inflation x2, patronizing title      |
| action-top-up-modal.tsx           | "Best Value" badge (minor)                  |
| action-dashboard.tsx              | Pedagogical subheading, compressed CTA      |
| auto-refill-settings.tsx          | Redundant label description                 |
| alert-settings-card.tsx           | "assisted actions" inconsistent noun phrase |
| ai-transparency-card.tsx          | None                                        |
| action-warning-banner-wrapper.tsx | None                                        |
| alert-settings-skeleton.tsx       | None                                        |
