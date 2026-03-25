# Phase 2: Cross-Cutting Patterns (Group 14)

**Slices analyzed**: 12 (pricing components `apps/marketing/src/components/pricing/` slices 156-157, shared marketing components slice 158, marketing config/lib slices 158-159, internal docs `docs/` slice 160, UI atoms `packages/ui/src/atoms/` slices 161-166)

---

## Pattern 1: Generic recommendation copy that ignores available data

**Slices affected**: apps-marketing-src-components-pricing-1 (action-topup-section.tsx), apps-marketing-src-components-pricing-2 (usage-calculator.tsx)

Both pricing components generate recommendation strings that use vague filler phrases despite having computed data available in the same scope:

| File                     | Phrase                                   | Data available but unused      |
| ------------------------ | ---------------------------------------- | ------------------------------ |
| action-topup-section.tsx | "Quick refill for light usage"           | Pack size and action count     |
| action-topup-section.tsx | "Best value for regular users"           | Price-per-action ratio         |
| action-topup-section.tsx | "Maximum capacity for heavy usage"       | Actual capacity number         |
| usage-calculator.tsx     | "Your usage fits comfortably in Starter" | Computed buffer percentage     |
| usage-calculator.tsx     | "Pro gives you room to grow"             | Computed buffer percentage     |
| usage-calculator.tsx     | "Max tier for heavy users"               | Exact action count from inputs |

**Systemic issue**: The pricing calculator and top-up section both compute specific numbers (buffer percentages, action counts, price-per-action) but then present recommendation text using generic marketing language. The data is available in the same `useMemo` block or adjacent constants. This is a missed opportunity to make the pricing experience feel data-driven, consistent with SlopWeaver's "provable" positioning.

**Suggested fix**: Replace each generic recommendation string with a template that uses the computed values. "Your usage fits comfortably in Starter" becomes "You have X% headroom in Starter based on your inputs." "Pro gives you room to grow" becomes "Pro gives you X% buffer at your current usage."

---

## Pattern 2: "Ready to get started?" CTA headline recurs in pricing

**Slices affected**: apps-marketing-src-components-pricing-1 (pricing-cta-section.tsx), also documented in Groups 12 and 13

The pricing CTA section uses "Ready to get started?" as its headline. This is the same stock SaaS CTA heading flagged in the features page (Group 12) and integration template (Group 13). Across all three groups, this phrase appears in at least three distinct CTA contexts.

**Cross-group pattern**: This is now confirmed as a site-wide CTA problem, not an isolated occurrence. Every major page section (features, integrations, pricing) closes with the same generic heading.

**Suggested fix**: Each CTA should connect to the content that precedes it. For pricing: "Start your trial. Watch your acceptance rate from day one." The body copy beneath the pricing CTA is already specific and good; the headline is the weak point.

---

## Pattern 3: Unsupported statistical claims at early-access scale

**Slices affected**: apps-marketing-src-components-pricing-1 (action-topup-section.tsx)

The top-up section states "Most users never run out of their monthly usage." At early access with a small user base, "most users" is an unverifiable statistical claim. This is the only instance found across all three groups, but it is worth flagging because overclaiming with implied statistics is a pattern that erodes credibility.

**Suggested fix**: Remove the claim entirely and replace with a factual statement: "Top-ups are available if you need extra capacity."

---

## Pattern 4: Retired tagline form ("AI that learns you") in pricing pill badge

**Slices affected**: apps-marketing-src-components-pricing-1 (pricing-cards-section.tsx), docs.md (CONTENT-PLAYBOOK.md gap)

The pricing cards section renders a pill badge with "AI that learns you," which is the shortened form of the canonical tagline "The AI that proves it's learning you." The shortened form drops the critical word "proves," which is the entire differentiator. The CONTENT-PLAYBOOK.md does not explicitly address shortened tagline forms, making this a guidance gap.

**Cross-reference**: The internal docs audit (slice 160) identified this same gap in the playbook: "AI that learns you" is not explicitly banned or addressed, though dropping "proves" defeats the positioning.

**Suggested fix**: Update the pill badge to "Provable learning" or the full tagline. Add explicit guidance to CONTENT-PLAYBOOK.md that shortening the tagline to remove "proves" is not acceptable.

---

## Pattern 5: Internal docs use structures they ban for external copy

**Slices affected**: docs.md (POSITIONING.md, CONTENT-PLAYBOOK.md)

POSITIONING.md uses two patterns that CONTENT-PLAYBOOK.md bans in external copy:

1. **Negative parallelism** (line 11): "Not 'unified inbox.' Not 'AI assistant.' Not 'productivity tool.'" matches the banned "Not X. Not Y. Just Z." pattern.
2. **Bold-first bullet lists** (lines 130-140): The Top 10 Differentiators section uses the bold-keyword-first list structure that the playbook bans as a "structural AI tell."

**Systemic issue**: An agent quoting these lines into marketing copy would violate the playbook. The distinction between "internal reference format" and "external copy format" is not stated in either document.

**Suggested fix**: Add inline notes in POSITIONING.md that these structures are internal reference formatting only and must not appear in external copy. Alternatively, restructure the internal docs to avoid normalizing banned patterns.

---

## Pattern 6: FAQ subheadline filler

**Slices affected**: apps-marketing-src-components-pricing-1 (faq-section.tsx)

The FAQ section subheadline reads "Everything you need to know about usage and pricing," which is the most common FAQ subheadline on the internet. The FAQ answers themselves are specific and reference real contract values. The subheadline undersells them.

**Suggested fix**: Replace with something specific: "How actions are counted, what rolls over, and what the trial includes."

---

## Pattern 7: UI atoms are completely clean

**Slices affected**: packages-ui-src-atoms-1 through packages-ui-src-atoms-5, packages-ui-src-atoms-icons-1

All 30+ UI atom components audited across six slices contain zero trope violations. These are structural primitives (buttons, inputs, badges, icons, layout components) with no hardcoded marketing copy. The only text strings are functional: "Loading" for screen readers, status labels like "Draft"/"Open"/"Closed," and aria-labels.

**Positive pattern**: The UI layer correctly delegates all copy responsibility to consuming components via props. No tropes can enter through atoms. This architecture is sound and should be preserved.

---

## Summary

| Pattern                                            | Severity | Slices               | Fix scope                          |
| -------------------------------------------------- | -------- | -------------------- | ---------------------------------- |
| Generic recommendation copy ignoring computed data | Medium   | 2                    | Two-file edit (template strings)   |
| "Ready to get started?" CTA (cross-group)          | Medium   | 1 (+ 2 other groups) | Site-wide CTA decision             |
| Unsupported "most users" statistical claim         | High     | 1                    | Single-line removal                |
| Retired tagline form without "proves"              | Medium   | 1 (+ playbook gap)   | Pill badge edit + playbook update  |
| Internal docs normalize banned patterns            | Low      | 1                    | Add inline notes to POSITIONING.md |
| FAQ subheadline filler                             | Low      | 1                    | Single-line edit                   |
| UI atoms are clean (positive)                      | N/A      | 6                    | No action needed                   |
