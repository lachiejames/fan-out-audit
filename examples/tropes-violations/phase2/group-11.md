# Phase 2: Cross-Cutting Patterns (Group 11)

**Slices analyzed**: 12 (pages: triage; pages root; shared/context; shared/platforms; app root; docs: AGENTS.md, CLAUDE.md, mdx-components.tsx; docs/src/app slices 1-4)

---

## Pattern 1: Fractal summary in closing callouts (docs site epidemic)

**Slices affected**: docs-src-app-2 (analytics, behavioral, contacts, inbox, knowledge-sources, meeting-prep), docs-src-app-3 (morning-brief, queue, search, triage, tasks) (2 of 12 input slices, but 11 individual doc pages)

Nearly every documentation page ends with a callout box that re-explains what the preceding section just covered. The closing callout adds no new information and follows a predictable formula: "[Feature] improves as the AI learns more / the more platforms you connect / over time."

| Doc page          | Closing callout text                                                                  |
| ----------------- | ------------------------------------------------------------------------------------- |
| analytics         | "Even early numbers show the AI is working and learning"                              |
| behavioral        | "The behavioral fingerprint is the foundation of all AI personalization"              |
| contacts          | "Entity resolution powers the AI's per-recipient style matching"                      |
| inbox             | "Every message you read, reply to, or archive feeds the AI's understanding"           |
| knowledge-sources | "Knowledge Sources accelerate the AI's learning by providing context..."              |
| meeting-prep      | "The more platforms connected and the more messages synced, the richer the briefings" |
| morning-brief     | "The more platforms you connect and the more knowledge sources you import..."         |
| queue             | "Every approval, edit, or rejection you make in the Queue trains the AI"              |
| search            | "Search results improve as the AI processes more of your data"                        |
| triage            | "Override learning is one of the most direct ways to train the AI"                    |
| tasks             | "Completing AI-extracted tasks contributes to your time-saved metric"                 |

**Systemic issue**: This is the single most pervasive trope pattern in the codebase. The callouts are all variations of "more data = better AI" and do not contribute page-specific information. They read as AI-generated summary padding.

**Suggested fix**: Remove the closing callouts entirely, or replace them with a single forward-link to Analytics ("See the impact in Analytics") without restating the learning loop.

---

## Pattern 2: Negative parallelism -- defining features by what they are not

**Slices affected**: docs-src-app-2 (behavioral, meeting-prep), docs-src-app-3 (morning-brief, triage) (2 of 12)

Multiple doc pages define features by contrasting them with bad alternatives rather than describing what the feature does directly:

| Doc page      | Text                                                                      | Issue                                             |
| ------------- | ------------------------------------------------------------------------- | ------------------------------------------------- |
| behavioral    | "Unlike a simple writing style model, it captures the full context..."    | Defines by contrast with unnamed inferior product |
| meeting-prep  | "This is not a generic summary."                                          | Negation before the positive claim                |
| morning-brief | "to avoid the generic, filler-heavy summaries that most AI tools produce" | Competitive swipe framing                         |
| triage        | "rather than reacting to messages throughout the day"                     | Implicit criticism of user's current behavior     |

**Systemic issue**: Negative parallelism is a flagged AI writing pattern. The positive claim always follows the negative opener, making the negative sentence redundant. Delete the negative sentence in each case; the positive claim that follows is strong enough to stand alone.

---

## Pattern 3: "More platforms = better AI" as a universal pitch

**Slices affected**: docs-src-app-2 (knowledge-sources), docs-src-app-3 (morning-brief, search, integrations) (2 of 12)

The statement "the more platforms/data/messages you connect/sync, the better/richer/more accurate the AI becomes" appears in some form on nearly every doc page. Beyond the closing callouts (Pattern 1), it is woven into body paragraphs as well:

| Doc page          | Variant                                                                       |
| ----------------- | ----------------------------------------------------------------------------- |
| knowledge-sources | "instead of waiting for it to discover everything from your messages"         |
| integrations      | "more platforms mean a richer behavioral fingerprint, better draft replies"   |
| search            | "the more messages and knowledge sources you have, the better results become" |

**Systemic issue**: This is a single idea restated across the entire docs site. It should be stated once (perhaps on the quick-start or integrations hub page) and linked to from other pages, not repeated on every feature page.

---

## Pattern 4: Pedagogical voice and signposted transitions

**Slices affected**: docs-src-app-1 (quick-start), docs-src-app-2 (analytics, knowledge-sources) (2 of 12)

Several doc pages use instructional preambles and transition sentences that announce what the page is about to do, rather than just doing it:

| Doc page          | Text                                                                  | Issue                                                     |
| ----------------- | --------------------------------------------------------------------- | --------------------------------------------------------- |
| quick-start       | "Get started with SlopWeaver quickly. Follow this guide..."           | Tells the reader they will follow a guide (they know)     |
| quick-start       | "Now that you're set up, explore these features:"                     | Signposted transition; the heading suffices               |
| analytics         | "Here is what to expect as the AI learns your patterns"               | "Here is what to expect" is a textbook pedagogical opener |
| knowledge-sources | "instead of waiting for it to discover everything from your messages" | Inflated framing of status quo pain                       |

**Systemic issue**: These are low-value filler sentences that pad page length without adding information. The fix is deletion in every case.

---

## Pattern 5: Ornate nouns and marketing metaphors in product docs

**Slices affected**: docs-src-app-2 (behavioral, keyboard-shortcuts), docs-src-app-3 (integrations, morning-brief) (2 of 12)

Several doc pages use metaphorical nouns that inflate what they describe:

| Doc page           | Text                                       | Issue                              |
| ------------------ | ------------------------------------------ | ---------------------------------- |
| behavioral         | "foundation of all AI personalization"     | "Foundation" used metaphorically   |
| keyboard-shortcuts | "your gateway to everything in SlopWeaver" | "Gateway" for a search bar         |
| integrations       | "feeds the AI's learning engine"           | "Learning engine" for the AI model |
| morning-brief      | "delivers a brief that respects your time" | "Respects your time" is a cliche   |

**Suggested fix**: Replace metaphorical nouns with direct descriptions. "Gateway" becomes "search and run any action." "Learning engine" becomes "the AI." "Respects your time" becomes "skips filler."

---

## Pattern 6: False precision in metrics claims

**Slices affected**: docs-src-app-3 (tasks) (1 of 12)

`tasks/page.tsx` states "Each completed TODO saves an estimated 5 minutes of manual organization time." This is a made-up number presented as a measurement. In a product that sells itself on provable, real metrics, fabricated time-savings figures undermine credibility.

This is a single occurrence but a high-severity finding given the product's positioning around measurable learning.

**Suggested fix**: Remove the specific figure, or document the methodology behind the estimate.

---

## Pattern 7: Pop-culture reference reused across pages

**Slices affected**: docs-src-app-2 (keyboard-shortcuts), docs-src-app-3 (search) (2 of 12)

"One shortcut to rule them all" (Lord of the Rings reference) appears on both the keyboard shortcuts page and the search page. Using the same quip twice dilutes both instances. Remove one occurrence.

---

## Pattern 8: PaywallContext as a cross-cutting trope source

**Slices affected**: shared/context (PaywallContext) (1 of 12, but affects many pages)

`PaywallContext.tsx` contains four findings that surface across the entire app whenever an upgrade prompt is triggered:

| Text                                     | Issue                           |
| ---------------------------------------- | ------------------------------- |
| "This feature is unlocked on paid plans" | "Unlocked" gate-removal framing |
| "Upgrade for unlimited usage" (x2)       | "Unlimited" likely inaccurate   |
| "Upgrade anytime. Cancel anytime."       | Stock SaaS reassurance fallback |

Because this file is a React context consumed by many pages, these strings have outsized user-visible impact. Fixing this single file addresses upgrade copy across the entire app.

---

## Pattern 9: App pages and internal docs are consistently clean

**Slices affected**: triage page, app root (App.tsx, main.tsx), shared/platforms, docs AGENTS.md, docs CLAUDE.md, docs mdx-components.tsx (6 of 12)

Six input slices had zero findings. The triage page, app bootstrap files, shared platform utilities, and internal developer docs are all clean. The triage page in particular is noted as well-written: concrete state descriptions, direct CTAs, no filler. Internal docs are operational references with no prose to audit.

The trope concentration is overwhelmingly in two areas:

1. **Public docs site** (`apps/docs/src/app/`) -- fractal summaries, negative parallelism, pedagogical voice
2. **Settings/AI profile components** (`apps/app/src/components/settings/`, `apps/app/src/pages/ai/`) -- AI-magic framing, vague empty states

Auth flows, task management, and infrastructure code remain clean.
