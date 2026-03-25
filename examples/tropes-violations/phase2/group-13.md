# Phase 2: Cross-Cutting Patterns (Group 13)

**Slices analyzed**: 12 (marketing integrations `apps/marketing/src/app/integrations/` slices 144-146, marketing pages `apps/marketing/src/app/` pricing/privacy/security/support/terms slices 147-152, marketing root slice 138, marketing components blog/marketing slices 153-155)

---

## Pattern 1: "Better Context" as a placeholder use case title across integrations

**Slices affected**: apps-marketing-src-app-integrations-1 (Google Docs), apps-marketing-src-app-integrations-2 (Jira, Linear), apps-marketing-src-app-integrations-3 (Microsoft Teams, Slack)

"Better Context" appears as a use case section title in 6+ integration configs in `_config.tsx`. It has become a templated placeholder that was never individualized. The descriptions beneath each "Better Context" heading are usually specific and good. The title itself is generic filler that could apply to any integration.

**Systemic issue**: The use case titles in `_config.tsx` were likely written with a template and never revisited. "Better Context" joins "Never Miss Deadlines" (Asana, Gmail), "Unified Communication" (Asana, Gmail), and "Stay Ahead of the Day" (both calendar integrations) as recurring generic headings.

**Suggested fix**: Replace each with an integration-specific title derived from the description beneath it. For Google Docs: "Source material close at hand." For Slack: "Chat history next to inbox and tasks." For Jira: "Issue history without the tab switch."

---

## Pattern 2: "Get Started Free" / "Ready to get started?" as template-level CTA boilerplate

**Slices affected**: apps-marketing-src-app-integrations-1 (template, all 17 integration pages), apps-marketing-src-components-marketing-1 (cta-section.tsx, marketing-nav.tsx), apps-marketing-src-components-marketing-2 (marketing-nav.tsx)

"Get Started Free" appears as a button label on every integration page (twice per page via the template), in the CTA section, and in the navigation bar. "Ready to get started?" appears as the CTA heading on the features page (Group 12) and the pricing CTA section (Group 14). Combined, these two phrases account for the majority of CTA touchpoints across the entire marketing site.

**Systemic issue**: This is a product-level decision, not a per-page copy fix. The labels are invisible due to ubiquity. They contribute to making the site feel templated despite strong body copy.

**Suggested fix**: Evaluate whether contextual CTAs would work better. For integration pages: "Connect [Platform]." For the pricing CTA: "Start your trial. Watch your acceptance rate from day one." For the nav: "Get Started" may be acceptable given space constraints.

---

## Pattern 3: Self-applied virtue labels in privacy, security, and trust contexts

**Slices affected**: apps-marketing-src-app-privacy (privacy/page.tsx), apps-marketing-src-app-security (security/page.tsx), apps-marketing-src-components-marketing-1 (cta-section.tsx)

Three files independently use self-declared trust/privacy labels that make claims the surrounding content must earn:

| File              | Phrase                                    | Problem                                     |
| ----------------- | ----------------------------------------- | ------------------------------------------- |
| privacy/page.tsx  | "takes your privacy seriously"            | Most cliched privacy opener in existence    |
| privacy/page.tsx  | "Privacy-respecting analytics"            | Self-applied label; name the tool instead   |
| privacy/page.tsx  | "industry-standard practices"             | Vague minimum-bar claim before good bullets |
| security/page.tsx | "privacy-first design" (meta description) | Self-applied marketing label                |
| cta-section.tsx   | "Your data stays yours"                   | Privacy-reassurance cliche                  |

**Systemic issue**: In each case, the content that follows is specific and credible (encrypted DB, TLS, AES-256-GCM tokens, RLS). The introductory phrases undercut the specifics by reverting to generic reassurance language. The pattern is: good bullets prefaced by a weak trust-claim opener.

**Suggested fix**: Remove the self-applied labels and let the specific technical facts lead. For privacy: "Your data is stored with the following security controls:" then bullets. For the CTA trust signal: "No model training on your data" or "OAuth tokens encrypted, data not sold."

---

## Pattern 4: "we'll" corporate plural on a solo-developer product

**Slices affected**: apps-marketing-src-app-support (support/page.tsx, support-form.tsx)

The support page correctly establishes "Real help from the person who built this" and frames SlopWeaver as a solo product. Then the form subtitle and success message both use "we'll get back to you," reverting to corporate plural. The same "we'll" inconsistency appears in the form component's success state.

**Suggested fix**: Replace "we'll" with "I'll" or reframe to "You'll hear back within 1 business day" to match the solo-founder voice established on the same page.

---

## Pattern 5: Weak-verb hedging in integration hero descriptions

**Slices affected**: apps-marketing-src-app-integrations-1 (Facebook Messenger), apps-marketing-src-app-integrations-2 (LinkedIn)

Two integration hero descriptions use hedged verb constructions:

| Integration        | Phrase                                          | Problem                                  |
| ------------------ | ----------------------------------------------- | ---------------------------------------- |
| Facebook Messenger | "SlopWeaver helps surface what needs attention" | "helps surface" hedges twice             |
| LinkedIn           | "SlopWeaver helps you shape copy in context"    | "helps you shape" is passive and awkward |

**Systemic issue**: These are the only two instances in the integration suite, but they share the same "helps [verb]" construction that weakens the claim. Either SlopWeaver does the thing or it does not.

**Suggested fix**: Remove "helps" and state the action directly. "SlopWeaver surfaces what needs attention first." "Connect LinkedIn to draft posts from your notes and launch plans."

---

## Pattern 6: "Human-in-the-loop" and "Proactive actions" as AI industry jargon in marketing

**Slices affected**: apps-marketing-src-components-marketing-1 (features-section.tsx)

Two feature titles use AI industry jargon as marketing labels: "Human-in-the-loop trust" and "Proactive actions." In both cases, the descriptions beneath the titles are specific and good. The titles themselves use insider terminology that means nothing to the average user.

**Suggested fix**: Rename to match what the descriptions actually say. "Human-in-the-loop trust" becomes "Approval queue, you decide what sends." "Proactive actions" becomes "Briefs, prep, and drafts before you ask."

---

## Pattern 7: "Made with care for busy professionals" footer cliche

**Slices affected**: apps-marketing-src-components-marketing-1 (marketing-footer.tsx)

The footer contains "Made with care for busy professionals," which combines two cliches: "made with care" (template footer sentiment) and "busy professionals" (target-audience placeholder used by every productivity tool). The tagline "The AI that proves it's learning you" already appears two lines above in the same footer, making this line redundant.

**Suggested fix**: Replace with something founder-specific: "Built by one developer in Melbourne." This is already true per the support page and is more distinctive.

---

## Pattern 8: Integrations index page uses vague AI claims despite strong per-integration copy

**Slices affected**: apps-marketing-src-app-integrations-1 (integrations/page.tsx)

The integrations index page description says "faster calibration, higher accuracy, better predictions" as a comma-stacked list of abstract AI marketing claims. The individual integration pages below it are mostly grounded and specific. The index page oversells relative to its children.

**Suggested fix**: Ground it with a specific example: "Connect Gmail and Slack and your AI learns whether you're formal with your manager in email but casual in chat. More tools, more signal, higher acceptance rate."

---

## Summary

| Pattern                                                  | Severity | Slices                               | Fix scope                   |
| -------------------------------------------------------- | -------- | ------------------------------------ | --------------------------- |
| "Better Context" placeholder title                       | High     | 5+ (6+ integrations in \_config.tsx) | Single file, 6+ edits       |
| "Get Started Free" / "Ready to get started?" boilerplate | Medium   | 4+ (template + CTA + nav)            | Product-level CTA decision  |
| Self-applied virtue labels (privacy/security/trust)      | High     | 3                                    | Three-file edit             |
| "we'll" corporate plural on solo product                 | Medium   | 2                                    | Two-file edit               |
| "helps [verb]" hedging in hero descriptions              | Low      | 2                                    | Two entries in \_config.tsx |
| AI jargon as feature titles                              | Medium   | 1 (features-section.tsx, 2 titles)   | Single-file edit            |
| "Made with care for busy professionals"                  | High     | 1                                    | Single-file edit            |
| Integrations index vague AI claims                       | Medium   | 1                                    | Single-file edit            |
