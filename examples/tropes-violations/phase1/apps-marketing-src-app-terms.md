# Trope Audit: apps/marketing/src/app/terms/page.tsx

Files audited:

- `apps/marketing/src/app/terms/page.tsx`

---

## Overall Assessment

Terms of Service pages are legal documents and are expected to use formal, templated language. The bar for "trope" here is different from marketing copy: the concern is language that is borrowed from generic SaaS legal templates without actually fitting the product. Most of this file is standard and appropriate. Findings are limited.

---

## Findings

### Finding 1: Section 2 description — "learn your work patterns"

- **Location**: lines 116–120
- **Offending text**: `"SlopWeaver is an AI productivity platform that integrates with third-party services (including Gmail, Slack, Linear, and others) to learn your work patterns, draft replies, extract action items, and track measurable improvement over time."`
- **Why it's flagged**: "AI productivity platform" is generic SaaS category language. In a legal document this matters less, but the description of service should use the product's specific language. "Learn your work patterns" is vaguer than the product's own positioning ("proves it's learning you" with metrics). This is the place to be precise about what the service actually does, for legal accuracy.
- **Suggested fix**: Minor. Could specify "builds behavioral profiles from connected tools and generates suggestions measured by acceptance rate, edit frequency, and time saved." But this is a low-priority legal copy issue.

### Finding 2: Section 9 — "suggested replies, suggested responses, and proactive recommendations"

- **Location**: lines 172–176
- **Offending text**: `"SlopWeaver uses artificial intelligence to generate content suggestions including suggested replies, suggested responses, and proactive recommendations."`
- **Why it's a trope**: "Suggested replies" and "suggested responses" mean the same thing and are listed as separate items. "Proactive recommendations" is vague AI-speak. In a legal context, redundancy is especially problematic because it implies there is a meaningful distinction where there is none.
- **Suggested fix**: `"SlopWeaver uses artificial intelligence to generate content suggestions, including draft replies, action item extractions, meeting briefs, and triage decisions."`

### Finding 3: "as is" disclaimer in Section 12

- **Location**: line 207
- **Offending text**: `'The service is provided &ldquo;as is&rdquo; without warranties of any kind.'`
- **Why it's noted**: Standard boilerplate, appropriate for Terms of Service. Not a marketing trope. Included here only to confirm it was reviewed and is expected.

---

## No marketing-copy tropes found

The metadata (title, description, OG tags) uses the expected "Terms of Service for SlopWeaver" language. No self-congratulatory phrases, empowerment language, or AI hype appear in this file.

---

## Summary

| #   | File           | Line(s) | Trope                                                                                 | Severity |
| --- | -------------- | ------- | ------------------------------------------------------------------------------------- | -------- |
| 1   | terms/page.tsx | 116–120 | "AI productivity platform" + "learn your work patterns" — generic service description | Low      |
| 2   | terms/page.tsx | 172–176 | "suggested replies, suggested responses" — redundant enumeration                      | Low      |
