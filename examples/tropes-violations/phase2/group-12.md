# Phase 2: Cross-Cutting Patterns (Group 12)

**Slices analyzed**: 12 (docs site `apps/docs/src/app/` slices 5-6, `apps/docs/src/components/` slices 133-134, `apps/docs/src/data/integrations/` slices 135-137, marketing `apps/marketing/src/app/` blog/changelog/delete-account/download/features slices 138-143)

---

## Pattern 1: Stock SaaS productivity phrases in integration descriptions

**Slices affected**: apps-docs-src-data-1 (jira, facebook-messenger), apps-docs-src-data-2 (monday, notion, microsoft-onedrive), apps-docs-src-app-5 (monday metadata)

The same family of worn-out productivity phrases repeats across integration intro copy in the docs data files:

| Phrase                                                  | Files                                    |
| ------------------------------------------------------- | ---------------------------------------- |
| "without context-switching" / "without switching tools" | jira.tsx, monday.tsx                     |
| "across your workspace"                                 | google-drive.tsx, microsoft-onedrive.tsx |
| "find what you need" (+ optional vague qualifier)       | google-docs.tsx, notion.tsx              |
| "stay on top of"                                        | jira.tsx                                 |
| "alongside your other work"                             | facebook-messenger.tsx                   |

**Systemic issue**: These phrases appear in the `intro` field of integration config data objects. Each integration was likely written independently, but the same filler phrases recur because they are the default language of SaaS productivity copy. They add no information and make each integration page sound interchangeable.

**Suggested fix**: Replace each with a concrete statement about what the integration actually does. The descriptions beneath these phrases are usually specific and good. Cut the filler clause and let the concrete description carry the weight.

---

## Pattern 2: "surface" as a verb for "show" in calendar integrations

**Slices affected**: apps-docs-src-data-1 (google-calendar), apps-docs-src-data-2 (microsoft-calendar)

Both calendar integration intros use "surface meeting context" as their opening verb construction. "Surface" as a verb is a product-writing cliche meaning "make visible" or "show." It appears in both calendar files with identical phrasing.

**Suggested fix**: Replace with "show" or a more concrete verb. "Connect Google Calendar to show meeting context in your inbox" is shorter and plainer.

---

## Pattern 3: "Ready to get started?" / "Get Started Free" as CTA boilerplate

**Slices affected**: apps-marketing-src-app-features (features/page.tsx), apps-docs-src-components-1 (integration page template indirectly)

The features page CTA section uses "Ready to get started?" as its heading and "Get Started Free" as its button label. This is the single most common SaaS CTA pattern. The features page itself contains strong, specific copy throughout, but the CTA section reverts to generic boilerplate, undermining the page's credibility.

This pattern recurs more broadly in Group 13 (integration pages, pricing CTA) and is flagged there as a systemic template-level issue.

**Suggested fix**: Replace with copy that connects to the page's thesis. For the features page: "Connect your tools. Track the improvement." or "See your AI's report card after the first week."

---

## Pattern 4: Fractal AI-learning callouts across docs pages

**Slices affected**: apps-docs-src-app-6 (cross-cutting finding across all 41 docs pages), apps-docs-src-components-1 (integration-page-template.tsx)

Nearly every feature page in the docs site ends with a callout saying (a) the AI learns from your interactions, and (b) check Analytics. The integration page template also includes a shared callout: "Data from this integration feeds the AI's learning." At scale, this creates a predictable, repetitive closing pattern that adds no page-specific information.

**Systemic issue**: The callout is rendered by `integration-page-template.tsx` for all 17 integration pages, and individual feature pages each add their own variant of the same message. The result is that a user browsing multiple pages sees the same sentiment restated dozens of times.

**Suggested fix**: Reserve the Analytics-link callout for pages where it is genuinely page-specific (e.g., the analytics page itself, the queue page where acceptance rate is central). Remove it from pages where it is a generic append.

---

## Pattern 5: "in your voice" as an AI writing cliche

**Slices affected**: apps-docs-src-app-6 (linkedin metadata), apps-docs-src-data-2 (linkedin.tsx)

"In your voice" appears in both the LinkedIn integration page metadata and the LinkedIn data config intro. This is one of the most overused phrases in AI writing tool marketing. Every AI email and writing tool uses it. For SlopWeaver, which actually has a measurable behavioral profile system, this generic phrase is a missed opportunity to say something more specific and credible.

**Suggested fix**: Replace with a concrete mechanism description: "The assistant drafts based on your past writing, which you review and approve before publishing."

---

## Pattern 6: Mock/fallback search excerpts with stock phrases

**Slices affected**: apps-docs-src-components-1 (docs-search.tsx)

The docs search component contains mock results with stock phrases: "Get up and running," "Master SlopWeaver," "Chat with your workspace assistant." These are visible in the search modal fallback UI.

**Systemic issue**: While low-visibility, these phrases normalize the kind of generic AI-product language the rest of the codebase avoids. Mock data should match the voice standards of real copy.

**Suggested fix**: Replace with specific descriptions: "Connect your tools and configure sync in under five minutes," "Keyboard shortcuts for inbox, queue, and search," "Ask questions, draft replies, and run actions from the AI panel."

---

## Pattern 7: Features page CTA undoes the page's credibility

**Slices affected**: apps-marketing-src-app-features (features/page.tsx)

The features page has strong, specific, differentiated copy throughout its body ("Your AI has a report card," "Writing patterns lose 5% confidence daily," "Morning briefs ban greetings, filler, and invented urgency"). The CTA section at the bottom reverts entirely to generic SaaS language: "Ready to get started?" + "watch your metrics improve over the first few weeks" + "Get Started Free." This pattern of strong body copy followed by weak CTA copy also appears in the marketing components (Group 13).

**Suggested fix**: The CTA should match the specificity of the body. Ground it in the same concrete claims the page already makes.

---

## Summary

| Pattern                                               | Severity | Slices                                | Fix scope                         |
| ----------------------------------------------------- | -------- | ------------------------------------- | --------------------------------- |
| Stock SaaS productivity phrases in integration intros | Medium   | 5                                     | Batch edit integration data files |
| "surface" as verb in calendar intros                  | Low      | 2                                     | Two-file edit                     |
| "Ready to get started?" / "Get Started Free" CTA      | Medium   | 2+ (systemic across groups)           | Template-level decision           |
| Fractal AI-learning callouts                          | Medium   | 17+ integration pages + feature pages | Template edit + per-page review   |
| "in your voice" cliche                                | High     | 2                                     | Two-file edit                     |
| Mock search excerpts with stock phrases               | Low      | 1                                     | Single-file edit                  |
| Strong body copy undermined by weak CTA               | Medium   | 1+ (cross-group)                      | CTA rewrite                       |
