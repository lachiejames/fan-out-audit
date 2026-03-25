# Trope Audit: apps/marketing/src/app/pricing/

Files audited:

- `apps/marketing/src/app/pricing/layout.tsx`
- `apps/marketing/src/app/pricing/page.tsx`

---

## layout.tsx

### Finding 1: Vague meta description — "fits your workflow"

- **Location**: line 10, `description` field
- **Offending text**: `"Transparent pricing for SlopWeaver. Choose the plan that fits your workflow. Free trial available."`
- **Why it's a trope**: "Fits your workflow" is a filler phrase that appears on nearly every SaaS pricing page. It says nothing specific about what the plans actually offer or who they're for.
- **Suggested fix**: Replace with something concrete, e.g., `"SlopWeaver pricing: Starter, Pro, and Max plans. Includes usage-based actions, message sync limits, and rollover. No credit card required to start."`

### Finding 2: OG/Twitter description — "for every workflow"

- **Location**: lines 12, 21
- **Offending text**: `"Transparent pricing for every workflow"`
- **Why it's a trope**: "Every workflow" is a universalizing platitude that tries to appeal to everyone and therefore appeals to no one. The word "transparent" as a self-descriptor is also a trust-claim the page has to earn rather than state.
- **Suggested fix**: Drop the claim, state the structure: e.g., `"Three plans, usage-based pricing, rollover included. See the numbers."`

---

## page.tsx

No user-visible copy is rendered directly in `page.tsx` — all copy lives in child components (`PricingCardsSection`, `FAQSection`, etc.) and in the `faqJsonLd` structured data block. The FAQ copy in the JSON-LD is audited below.

### Finding 3: FAQ — "How does usage work?" answer is adequate but buries the unit

- **Location**: lines 24–27, `faqJsonLd.mainEntity[0]`
- **Offending text**: `"Usage units cover reply and analysis actions (for example: quick chat is ..., a simple reply is ..., and a standard reply is ...). Your subscription includes a monthly usage allocation that resets each billing cycle."`
- **Why it's a trope**: The phrase "monthly usage allocation that resets each billing cycle" is bureaucratic filler — "resets each billing cycle" restates that it is monthly. Not a headline trope, but a wordiness flag.
- **Suggested fix**: `"Usage units cover reply and analysis actions — quick chat costs X, a simple reply costs Y, a standard reply costs Z. Each plan includes a monthly usage budget."`

### Finding 4: FAQ — "More synced messages means more data for the AI to learn from"

- **Location**: lines 79–83, `faqJsonLd.mainEntity[7]`
- **Offending text**: `"More synced messages means more data for the AI to learn from."`
- **Why it's a trope**: "More data for the AI to learn from" is a generic AI-hype phrase. The sync limit FAQ is otherwise factual and good; this sentence adds a marketing gloss that dilutes credibility.
- **Suggested fix**: Remove the sentence entirely. The preceding facts (5k/25k/100k limits by tier) are self-explanatory. Alternatively: `"Higher tiers give the AI more history to draw on when drafting replies and building your behavioral profile."`

---

## Summary

| #   | File       | Line(s) | Trope                                                       | Severity |
| --- | ---------- | ------- | ----------------------------------------------------------- | -------- |
| 1   | layout.tsx | 10      | "fits your workflow" — empty SaaS filler                    | Low      |
| 2   | layout.tsx | 12, 21  | "for every workflow" / self-proclaimed "Transparent"        | Low      |
| 3   | page.tsx   | 24–27   | "resets each billing cycle" — redundant bureaucratic phrase | Low      |
| 4   | page.tsx   | 79–83   | "more data for the AI to learn from" — generic AI hype      | Medium   |
