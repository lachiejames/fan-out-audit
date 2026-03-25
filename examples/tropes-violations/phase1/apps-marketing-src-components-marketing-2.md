# Trope Audit: apps/marketing/src/components/marketing/ (Part 2)

Files audited:

- `apps/marketing/src/components/marketing/marketing-nav.tsx`
- `apps/marketing/src/components/marketing/pain-points-section.tsx`

---

## marketing-nav.tsx

No marketing copy. The nav contains only link labels ("Features", "Pricing", "Download", "Blog", "Docs") and button labels ("Log In", "Get Started"). The only potentially trope-adjacent item:

### Finding 1: CTA button — "Get Started"

- **Location**: lines 65, 108
- **Offending text**: `"Get Started"` (desktop and mobile)
- **Why it's noted**: "Get Started" is the single most common SaaS CTA button label. It is completely generic. Flagged for consistency with the same finding in `cta-section.tsx`.
- **Suggested fix**: Given the nav context (space-constrained), this is lower priority than the hero/CTA section CTAs. A product decision is needed: either standardize on a specific label across all CTAs or accept "Get Started" for nav use. Minor.

---

## pain-points-section.tsx

### Finding 2: Section heading — "Sound familiar?"

- **Location**: line 33
- **Offending text**: `"Sound familiar?"`
- **Why it's a partial flag**: "Sound familiar?" is a very common rhetorical device used in SaaS pain-point sections to create false intimacy with the reader. It is an established pattern rather than a differentiated voice. That said, the pain points that follow are genuinely specific and the copy avoids other tropes.
- **Suggested fix**: Consider a more direct lead, e.g., `"The real problem with AI tools"` which connects directly to the subheadline. Or keep it — this is a moderate flag, not a strong violation, and the surrounding copy quality is high enough that it doesn't feel as hollow as it would in a weaker section.

### Finding 3: Pain point 01 — "trust the marketing"

- **Location**: lines 4–8
- **Offending text**: `'Every AI tool says it "learns your style." None of them show you the data. You just have to trust the marketing.'`
- **Assessment**: This is strong and specific. Naming competitors as hiding behind marketing language, while SlopWeaver shows data, is exactly the right framing. Not a trope — this is deliberate and earned.

### Finding 4: Pain point 04 — "AI that forgets everything"

- **Location**: lines 21–24
- **Offending text**: label: `"AI that forgets everything."`, description: `"You paste context every time. You retype preferences. The AI starts from scratch each session."`
- **Assessment**: Specific and accurate. Not a trope. The description avoids generic AI criticism language and describes a concrete behavior.

### Finding 5: Closing paragraph — "decide for yourself"

- **Location**: lines 57–60
- **Offending text**: `"SlopWeaver tracks its own accuracy and shows you the trend over time. You can see whether the AI is improving or stalling, and decide for yourself."`
- **Assessment**: "Decide for yourself" is occasionally a cliche but here it is earning its place — the entire section is about the absence of proof in competitors, so "decide for yourself" is the natural conclusion. Not a flag.

### Finding 6: Subheadline — "You can't tell if your AI is getting better or just generating slop"

- **Location**: lines 36–38
- **Offending text**: `"You can't tell if your AI is getting better or just generating slop. That's the real problem."`
- **Assessment**: This is the best copy in the section. Direct, uses the product name's own register ("slop"), and names the real problem without hedging. Not a trope — the opposite.

---

## Summary

| #   | File                    | Line(s) | Trope                                               | Severity |
| --- | ----------------------- | ------- | --------------------------------------------------- | -------- |
| 1   | marketing-nav.tsx       | 65, 108 | "Get Started" — ubiquitous CTA (minor, nav context) | Low      |
| 2   | pain-points-section.tsx | 33      | "Sound familiar?" — common rhetorical SaaS device   | Low      |
