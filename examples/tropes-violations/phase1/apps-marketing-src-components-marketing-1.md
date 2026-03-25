# Trope Audit: apps/marketing/src/components/marketing/ (Part 1)

Files audited:

- `apps/marketing/src/components/marketing/comparison-section.tsx`
- `apps/marketing/src/components/marketing/cta-section.tsx`
- `apps/marketing/src/components/marketing/download-section.tsx`
- `apps/marketing/src/components/marketing/features-section.tsx`
- `apps/marketing/src/components/marketing/hero-section.tsx`
- `apps/marketing/src/components/marketing/integration-logos.tsx` — FILE DOES NOT EXIST
- `apps/marketing/src/components/marketing/marketing-footer.tsx`
- `apps/marketing/src/components/marketing/marketing-header.tsx` — FILE DOES NOT EXIST

---

## comparison-section.tsx

### Finding 1: "What you can measure" heading is good — NOT a trope

- **Location**: line 41
- **Offending text**: `"What you can measure"`
- **Assessment**: This heading is specific to SlopWeaver's differentiator. It is not a generic AI trope. Keeping it.

### Finding 2: Comparison feature — "Drafts in your voice"

- **Location**: line 13 in `comparisons` array
- **Offending text**: `{ competitors: "partial", feature: "Drafts in your voice", slopweaver: true }`
- **Why it's a trope**: "In your voice" is one of the most overused AI writing assistant phrases. Every AI email tool uses it. Given that the comparison table is explicitly differentiating SlopWeaver from competitors, having "in your voice" as a feature row actually weakens the case — it's the same language competitors use.
- **Suggested fix**: Reframe around the measurable difference. e.g., `"Draft style calibrates from your correction history"` or `"Style accuracy measured by edit-per-draft rate"`. This uses SlopWeaver's actual language.

### Finding 3: "Most AI tools claim to learn your style but give you no way to verify"

- **Location**: lines 44–47
- **Offending text**: `"Most AI tools claim to learn your style but give you no way to verify. SlopWeaver ships an analytics page where every claim is checkable."`
- **Assessment**: This is strong and specific. Not a trope — it is the actual product differentiator stated plainly. No change.

### Finding 4: "Walled-garden AI" competitive framing

- **Location**: lines 30–34 in `competitiveFraming`
- **Offending text**: `"SlopWeaver connects 17 tools and builds context automatically from how you use them."`
- **Why it's noted**: "Builds context automatically" is generic AI-speak. The "17 tools" is specific. The phrase "from how you use them" is stronger than most but still slightly vague.
- **Suggested fix**: `"SlopWeaver connects 17 tools and builds a behavioral profile across all of them — one identity across email, Slack, and issue trackers."`

---

## cta-section.tsx

### Finding 5: "See the learning curve for yourself"

- **Location**: line 26
- **Offending text**: `"See the learning curve for yourself"`
- **Assessment**: This is specific to SlopWeaver's core claim. "Learning curve" has a double meaning (the traditional phrase + the actual metrics curve) that is intentional and on-brand. Not a trope.

### Finding 6: "Connect your tools and watch your metrics improve over the first few weeks"

- **Location**: lines 28–30
- **Offending text**: `"Connect your tools and watch your metrics improve over the first few weeks. Nothing sends without your approval."`
- **Assessment**: Good. Specific, concrete, and includes the key safety framing. Not a trope.

### Finding 7: Trust signal — "Your data stays yours"

- **Location**: line 10 in `trustSignals`
- **Offending text**: `{ icon: Shield, text: "Your data stays yours" }`
- **Why it's a trope**: "Your data stays yours" is a privacy-reassurance cliche used across SaaS. It is technically true of almost every SaaS product and therefore distinguishes nothing. It's the Shield icon equivalent of "we take privacy seriously."
- **Suggested fix**: Something SlopWeaver-specific: `"No model training on your data"` or `"Data never leaves your account"` or simply reference what the security page actually says: `"OAuth tokens encrypted, data not sold"`.

### Finding 8: CTA button — "Get Started Free"

- **Location**: line 39
- **Offending text**: `"Get Started Free"`
- **Why it's a trope**: "Get Started Free" appears on roughly 80% of SaaS CTAs. It is not wrong, but it is invisible due to ubiquity.
- **Suggested fix**: Something that connects to the product's proof claim: `"Start your free trial"` is slightly better. More on-brand: `"See your metrics"` or `"Connect your first tool"`. The level of change required here is a product decision, not just a copy decision.

---

## download-section.tsx

No marketing copy. The only user-visible text is platform labels ("macOS", "App Store", "Google Play"), descriptions ("Direct download (DMG)", "iOS on App Store", "Android on Google Play"), and the state label "Coming soon" for unavailable platforms. All functional, no trope risk.

**No findings.**

---

## features-section.tsx

### Finding 9: Section heading — "Watch your AI get measurably better"

- **Location**: line 42
- **Offending text**: `"Watch your AI get measurably better"`
- **Assessment**: Strong. "Measurably" is the key word — it is SlopWeaver's central claim. Not a trope.

### Finding 10: Feature description — "Proactive actions"

- **Location**: lines 17–23 in `features` array
- **Offending text**: title: `"Proactive actions"`, description: `"Morning briefs summarize what matters. Meeting prep assembles context from 4+ platforms. Draft replies appear before you open the message. TODOs extract themselves from conversations."`
- **Why it's a partial flag**: "Proactive actions" as a feature title is generic AI-speak. The description is actually specific and good. The title undersells the description.
- **Suggested fix**: Rename the feature to match what the description actually says, e.g., `"Briefs, prep, and drafts before you ask"` or `"Ahead-of-time assistance"`.

### Finding 11: Feature — "Human-in-the-loop trust"

- **Location**: lines 24–30 in `features` array
- **Offending text**: title: `"Human-in-the-loop trust"`
- **Why it's a trope**: "Human-in-the-loop" is AI industry jargon that has escaped into marketing copy. It is the kind of phrase that sounds technical-credible but means nothing to the average user. The description that follows is good: "Drafts show their reasoning and costs are transparent. Every action waits in an approval queue you control."
- **Suggested fix**: Rename the feature to the plain-language version: `"Approval queue — you decide what sends"` or `"Every action waits for your sign-off"`.

### Finding 12: "Every feature is built around a learning loop"

- **Location**: lines 44–47
- **Offending text**: `"Every feature is built around a learning loop. Connect your tools, let the AI observe, correct what it gets wrong, and watch the metrics improve."`
- **Why it's a partial flag**: "Learning loop" is slightly jargon-adjacent. The rest of the sentence is concrete and good.
- **Suggested fix**: `"Every feature feeds back into accuracy. Connect your tools, correct what the AI gets wrong, and track whether the numbers improve."`

---

## hero-section.tsx

### Finding 13: Hero headline — "AI that proves it's learning."

- **Location**: line 39
- **Offending text**: `"AI that proves it's learning."`
- **Assessment**: This is the product's core tagline and is differentiated. Not a trope.

### Finding 14: Hero subheadline

- **Location**: lines 43–45
- **Offending text**: `"SlopWeaver connects Gmail, Slack, Linear, and 14 more tools. It learns your patterns. Track acceptance rates climbing, edits decreasing, and hours saved growing."`
- **Assessment**: Specific, concrete, metric-forward. The three outcomes (acceptance rates, edits, hours) are the right things to name. Not a trope.

### Finding 15: "Early access" badge

- **Location**: line 37
- **Offending text**: `"Early access"`
- **Assessment**: Factually accurate label. Not a trope.

### Finding 16: Metrics widget — "Your AI is learning"

- **Location**: line 113
- **Offending text**: `"Your AI is learning"`
- **Why it's noted**: As a widget label this is fine — it's the visual proof of the claim. In context it works. Not a flag.

### Finding 17: "47 writing patterns learned" / "Calibrating daily"

- **Location**: lines 164–165
- **Offending text**: `"47 writing patterns learned"` and `"Calibrating daily"`
- **Why it's a partial flag**: These are shown as mock data in the hero widget. "Calibrating daily" is slightly jargon-adjacent. "Writing patterns learned" is a meaningful metric but "patterns" is vague. If these become real product metrics, they'll need precise definitions. As marketing mock data they're acceptable.
- **Suggested fix**: If these numbers appear in real product UI, replace "writing patterns" with the actual metric name used in the codebase.

---

## marketing-footer.tsx

### Finding 18: "Made with care for busy professionals"

- **Location**: line 67
- **Offending text**: `"Made with care for busy professionals."`
- **Why it's a trope**: "Busy professionals" is a target-audience cliche used by every productivity tool ever made. "Made with care" is a sentiment that reads as template footer text. Neither phrase adds information or distinguishes the product.
- **Suggested fix**: Use the tagline already established for the product: `"The AI that proves it's learning you."` — which is already present in the brand column two lines above (line 31), making this footer line redundant. Alternatively, something founder-specific: `"Built by one developer in Melbourne."` which is already true per the support page.

---

## integration-logos.tsx and marketing-header.tsx

Both files do not exist at the specified paths. Cannot audit.

---

## Summary

| #   | File                   | Line(s) | Trope                                                             | Severity |
| --- | ---------------------- | ------- | ----------------------------------------------------------------- | -------- |
| 1   | comparison-section.tsx | 13      | "Drafts in your voice" — overused AI assistant phrase             | Medium   |
| 2   | comparison-section.tsx | 30–34   | "builds context automatically" — generic AI-speak                 | Low      |
| 3   | cta-section.tsx        | 10      | "Your data stays yours" — privacy cliche                          | Medium   |
| 4   | cta-section.tsx        | 39      | "Get Started Free" — ubiquitous CTA                               | Low      |
| 5   | features-section.tsx   | ~18     | "Proactive actions" — generic AI feature name                     | Medium   |
| 6   | features-section.tsx   | ~25     | "Human-in-the-loop trust" — AI industry jargon as marketing       | High     |
| 7   | features-section.tsx   | 44–47   | "learning loop" — jargon-adjacent                                 | Low      |
| 8   | features-section.tsx   | 44–47   | "Every feature is built around a learning loop"                   | Low      |
| 9   | marketing-footer.tsx   | 67      | "Made with care for busy professionals" — generic audience cliche | High     |
