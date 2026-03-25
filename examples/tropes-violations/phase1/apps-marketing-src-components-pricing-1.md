# Trope Audit: apps/marketing/src/components/pricing (Slice 156)

Files audited:

- action-topup-section.tsx
- billing-toggle.tsx
- comparison-table.tsx
- faq-section.tsx
- pricing-card.tsx (actual file; feature-list.tsx, plan-card.tsx, pricing-hero.tsx, pricing-section.tsx in the slice spec do not exist)
- pricing-cards-section.tsx (actual file)
- pricing-cta-section.tsx (actual file)
- social-proof-section.tsx (actual file)

Note: The slice specified feature-list.tsx, plan-card.tsx, pricing-hero.tsx, and pricing-section.tsx, none of which exist. The actual pricing directory contains pricing-card.tsx, pricing-cards-section.tsx, pricing-cta-section.tsx, and social-proof-section.tsx, which are audited here instead.

---

## Findings

### action-topup-section.tsx

**Line 13** - `description: "Quick refill for light usage"`
Bland filler. "Light usage" says nothing specific. Could name actual actions or a count.

**Line 22** - `description: "Best value for regular users"`
"Best value" is a superlative claim with no supporting data in context. The badge label "Best Value" on line 66 is fine (it's a badge, not copy), but as a description it's thin.

**Line 31** - `description: "Maximum capacity for heavy usage"`
"Maximum capacity" is borderline corporate speak. Fine as a descriptor but reads like spec sheet language.

**Line 158** - `"Most users never run out of their monthly usage."`
Overclaiming without a source. "Most users" is a vague claim that cannot be verified at early access stage with a small user base. Should be removed or replaced with something honest ("Top-ups are available if you need extra capacity" without the unsupported statistical claim).

---

### billing-toggle.tsx

No violations. UI-only toggle with "Monthly" / "Annual" / "Save 17%" labels. Clean.

---

### comparison-table.tsx

**Line 112** - `"More data and actions means faster, more accurate learning"`
This is a claim presented as fact in the section subheadline. It's directionally true and is the core product thesis, but "more accurate" is an unqualified superlative. Consider: "More data means more to learn from. More actions mean more signals." The current phrasing is borderline; not a hard violation but worth tightening.

No banned phrases detected. Table content is factual and specific.

---

### faq-section.tsx

**Line 53** - `"Everything you need to know about usage and pricing"`
Classic AI-generated subheadline filler. Adds no information. Every FAQ section on the internet says this. Cut it or replace with something specific, for example: "How actions are counted, what rolls over, and what the trial includes."

No other violations. FAQ answers are specific, reference real contract values, and avoid banned phrases.

---

### pricing-card.tsx

**Line 25** (plans in pricing-cards-section.tsx, description for Starter) - `"Start training your AI. See your first acceptance rate after a week."`
Clean, specific, on-brand. No violations.

**Line 44** - `"Full behavioral fingerprinting. Watch your acceptance rate climb as the AI learns your patterns across all 17 tools."`
Solid. Specific and provable-learning-aligned.

**Line 65** - `"Maximum data, fastest learning. Extended history and the most capable models for high-volume workflows."`
"Fastest learning" is a superlative without a baseline. Not egregious in context (it is the highest tier), but "faster learning" (comparative) would be more defensible.

No structural AI tells. No banned phrases.

---

### pricing-cards-section.tsx

**Line 104** - `<span className="font-medium text-foreground text-sm">AI that learns you</span>`
This pill label is fine as a badge, but "AI that learns you" is the retired tagline form. The canonical tagline is "The AI that proves it's learning you." Consider updating the pill to match: "The AI that proves it's learning you" or just "Provable learning."

**Line 110-112** - `"More data means faster learning. More actions mean more AI suggestions you can accept, reject, or edit, and each decision teaches the AI your preferences. Choose the tier that fits your workflow."`
Good. Clear, specific, no banned phrases.

---

### pricing-cta-section.tsx

**Line 16-17** - `"Ready to get started?"`
Stock filler headline. Every SaaS pricing page ends with this. It wastes the CTA moment. Replace with something specific to the provable learning positioning, for example: "Start your trial. Watch your acceptance rate from day one."

**Line 18-21** - `"{TRIAL_CONFIG.durationDays}-day trial with up to {TRIAL_CONFIG.actions} actions. Connect your tools, watch the AI learn your patterns, and track your acceptance rate from day one. No credit card required."`
The body is good and specific. The headline is the weak point.

---

### social-proof-section.tsx

**Line 29** - `"SlopWeaver is in early access. Built by a solo developer who uses it daily."`
Clean, honest, no violations. Aligns with early access framing guidance.

No violations in trust badges (Encrypted at rest, Row-level security, US hosted). Factual and brief.

---

## Summary

| File                      | Violations                                                 |
| ------------------------- | ---------------------------------------------------------- |
| action-topup-section.tsx  | 2 (unsupported "most users" claim; weak pack descriptions) |
| billing-toggle.tsx        | 0                                                          |
| comparison-table.tsx      | 1 (borderline unqualified superlative in subheadline)      |
| faq-section.tsx           | 1 (filler subheadline)                                     |
| pricing-card.tsx          | 1 (borderline "fastest learning" superlative)              |
| pricing-cards-section.tsx | 1 (retired tagline form in pill badge)                     |
| pricing-cta-section.tsx   | 1 (stock "Ready to get started?" headline)                 |
| social-proof-section.tsx  | 0                                                          |

**Priority fixes:**

1. `action-topup-section.tsx` line 158: Remove "Most users never run out" (unsupported claim at early access scale).
2. `pricing-cta-section.tsx` line 17: Replace "Ready to get started?" with a provable-learning-anchored headline.
3. `pricing-cards-section.tsx` pill badge: Align with current canonical tagline.
4. `faq-section.tsx` line 53: Cut or replace the filler subheadline.
