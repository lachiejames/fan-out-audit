# Tropes Audit: apps/docs/src/app — Slice 3

**Files audited**:

- `src/app/morning-brief/page.tsx` (Morning Briefs)
- `src/app/queue/page.tsx` (Queue)
- `src/app/search/page.tsx` (Search)
- `src/app/triage/page.tsx` (Triage)
- `src/app/voice/page.tsx` (Voice)
- `src/app/settings/page.tsx` (Settings)
- `src/app/tasks/page.tsx` (Tasks)
- `src/app/integrations/page.tsx` (Integrations hub)

---

## src/app/morning-brief/page.tsx (Morning Briefs)

**User-facing prose** (selected passages):

> "Morning Briefs give you a concise summary of what happened overnight and what needs your attention today. The AI compiles activity from all connected platforms, prioritizes based on your behavioral fingerprint, and delivers a brief that respects your time. No filler, no invented urgency."

> "Morning briefs follow strict content rules to avoid the generic, filler-heavy summaries that most AI tools produce."

> "The brief does not start with 'Good morning' or 'Here's your daily summary.' It starts with the most important item."

> "Morning briefs use the same AI context as draft replies. The more platforms you connect and the more knowledge sources you import, the more relevant your briefs become."

**Findings**:

1. **Negative parallelism (multiple)**: The anti-slop section is built on negative parallelism — defining the feature by what it does not do:
   - "to avoid the generic, filler-heavy summaries that most AI tools produce" — defines the product by contrast with competitors
   - "No greetings or pleasantries" / "No invented urgency" / "No filler content" — these headings define by negation
   - "The brief does not start with 'Good morning'..." — defines by what it is not

   The anti-slop section has a legitimate purpose (it is documenting a real product rule), but the framing leans heavily on negative parallelism throughout. The section could open by stating what a brief _does_ contain before listing what it excludes.

2. **Fractal summary**: Closing callout — "The more platforms you connect and the more knowledge sources you import, the more relevant your briefs become." — same general principle stated in multiple other pages. No new information specific to morning briefs.

3. **Stakes inflation**: "to avoid the generic, filler-heavy summaries that most AI tools produce" — sweeping claim about "most AI tools." Not verifiable and functions as implicit competitive positioning rather than product documentation.

**Severity**: Low-medium. The negative parallelism in the anti-slop section is intentional (it's documenting a product decision), but the framing of the intro paragraph and closing callout could be cleaned up.

**Specific violations**:

- Intro: "delivers a brief that respects your time. No filler, no invented urgency." — "respects your time" is a cliche. Consider "delivers a brief that skips filler and invented urgency."
- Closing callout: drop or shorten; the "more platforms = better" principle doesn't need restating here.
- Anti-slop intro: "to avoid the generic, filler-heavy summaries that most AI tools produce" — remove the competitive comparison; say what the rules achieve, not what they're avoiding.

---

## src/app/queue/page.tsx (Queue)

**User-facing prose** (selected passages):

> "Every reply needs your approval before sending. Reply quality improves with your feedback, and you can accept, edit, or reject any suggestion."

> "SlopWeaver does not send messages on your behalf without explicit approval. Every reply stays in your queue until you click 'Send' or 'Reject'."

> "Every approval, edit, or rejection you make in the Queue trains the AI. Over time, suggestions become more accurate and confidence scores increase. Track your acceptance rate in Analytics."

**Findings**:

1. **Fractal summary**: Closing callout — "Every approval, edit, or rejection you make in the Queue trains the AI. Over time, suggestions become more accurate and confidence scores increase." — re-explains the style-learning section that precedes it. No new information.

**Severity**: Low.

---

## src/app/search/page.tsx (Search)

**User-facing prose** (selected passages):

> "One shortcut to rule them all: ⌘K. Search messages, contacts, tasks, and more across your connected integrations."

> "Search results improve as the AI processes more of your data. Semantic search uses embeddings trained on your content, so the more messages and knowledge sources you have, the better results become."

**Findings**:

1. **"One shortcut to rule them all"**: Pop-culture reference (Lord of the Rings). This phrase also appears in `keyboard-shortcuts/page.tsx`. Using it twice across the docs weakens both instances. One should be removed.

2. **Fractal summary**: Closing callout — "Search results improve as the AI processes more of your data." — generic "more data = better AI" statement repeated across many pages. No search-specific information.

**Severity**: Low.

---

## src/app/triage/page.tsx (Triage)

**User-facing prose** (selected passages):

> "Triage is a focused, session-based workflow for processing your inbox efficiently. The AI classifies each item with a suggested action and confidence score. You approve, override, or skip with keyboard shortcuts. Every decision you make trains the AI. After 3 consistent overrides on the same type of item, the AI creates an automatic rule."

> "Triage sessions have a beginning and end. You start a session, process items until done (or pause), and see a summary of what you accomplished. This structure helps you clear your inbox in focused blocks rather than reacting to messages throughout the day."

> "Override learning is one of the most direct ways to train the AI. Consistent corrections produce automatic rules, which improve both triage accuracy and your acceptance rate in Analytics."

**Findings**:

1. **Fractal summary**: The callout "Override learning is one of the most direct ways to train the AI. Consistent corrections produce automatic rules..." recaps the section it follows rather than adding new information.

2. **"rather than reacting to messages throughout the day"**: Mild negative parallelism and slight stakes inflation. Defines the feature by contrast with an unstated bad behavior pattern.

**Severity**: Low.

---

## src/app/voice/page.tsx (Voice)

**User-facing prose** (selected passages):

> "Use voice input, voice output, or both. SlopWeaver keeps the same Anthropic tool-calling agent as the brain and uses ElevenLabs for speech-to-text and text-to-speech."

**Findings**: None. The page is almost entirely structured lists and configuration tables. The one prose sentence is direct and factual. No flagged patterns.

---

## src/app/settings/page.tsx (Settings)

**User-facing prose** (selected passages):

> "Configure integrations, adjust sync behavior, manage notifications, and control your account. Settings are organized into 4 tabs for easy access."

> "Control how SlopWeaver works for you" (subheading)

> "Disconnecting a platform removes all synced data from SlopWeaver. This action cannot be undone."

> "Deleting your account is permanent. All messages, tasks, and indexed data will be erased. This action cannot be undone."

**Findings**:

1. **"organized into 4 tabs for easy access"**: "for easy access" is a hollow benefit claim. Cut it — "Settings are organized into 4 tabs" is sufficient.

**Severity**: Minimal.

---

## src/app/tasks/page.tsx (Tasks)

**User-facing prose** (selected passages):

> "AI extracts action items from your message threads. Each task includes priority, effort estimate, and source context."

> "SlopWeaver scans every message for action items and surfaces suggested tasks for you. You review, edit, or delete them as needed. AI suggests priority and effort, but you make the call."

> "AI analyzes each task to estimate how important it is and how long it will take. This helps you decide what to tackle first."

> "Completing AI-extracted tasks contributes to your time-saved metric. Each completed TODO saves an estimated 5 minutes of manual organization time, tracked in Analytics."

**Findings**:

1. **False precision**: "Each completed TODO saves an estimated 5 minutes of manual organization time" — this is a made-up number presented as a metric. Calling it "estimated" doesn't redeem it; it's an illustrative figure being treated as a real measurement. Either supply the actual calculation methodology or remove the specific number.

2. **Fractal summary**: "Completing AI-extracted tasks contributes to your time-saved metric." — restates analytics linkage that's already been covered throughout the docs.

**Severity**: Low-medium for item 1 (false precision in a metric-focused product is a credibility risk).

**Specific violation**:

- `tasks/page.tsx` closing callout: "Each completed TODO saves an estimated 5 minutes of manual organization time" — remove the specific figure or explain where it comes from.

---

## src/app/integrations/page.tsx (Integrations Hub)

**User-facing prose**:

> "17 integrations are available. Each connected platform feeds the AI's learning engine: more platforms mean a richer behavioral fingerprint, better draft replies, and more accurate triage suggestions. Track the impact of each integration in Analytics."

> "More integrations are in progress. Setup docs will publish as each one launches."

**Findings**:

1. **Ornate noun**: "feeds the AI's learning engine" — "learning engine" is a metaphorical noun inflating what is simply the AI model. "Each connected platform gives the AI more data to learn from" would be direct.

**Severity**: Low.

---

## Summary for Slice 3

| File                   | Findings                                                                                       |
| ---------------------- | ---------------------------------------------------------------------------------------------- |
| morning-brief/page.tsx | 3 findings: negative parallelism (intentional but overused), fractal summary, stakes inflation |
| queue/page.tsx         | 1 low finding: fractal summary in closing callout                                              |
| search/page.tsx        | 2 low findings: repeated pop-culture hook, fractal summary                                     |
| triage/page.tsx        | 2 low findings: fractal summary, negative parallelism                                          |
| voice/page.tsx         | None                                                                                           |
| settings/page.tsx      | 1 minimal finding: hollow benefit phrase                                                       |
| tasks/page.tsx         | 2 findings: false precision (low-medium), fractal summary                                      |
| integrations/page.tsx  | 1 low finding: ornate noun                                                                     |

**Priority violations to fix**:

1. `tasks/page.tsx` closing callout: Remove "Each completed TODO saves an estimated 5 minutes of manual organization time" — this is a made-up number in a product that sells on real metrics.
2. `morning-brief/page.tsx`: Remove the competitive swipe "generic, filler-heavy summaries that most AI tools produce"; replace with a direct statement of what the anti-slop rules achieve.
3. `morning-brief/page.tsx` intro: Replace "delivers a brief that respects your time" with concrete language.
4. `search/page.tsx`: Remove the second instance of "One shortcut to rule them all" (it also appears in keyboard-shortcuts).
5. `integrations/page.tsx`: Replace "feeds the AI's learning engine" with direct language.
