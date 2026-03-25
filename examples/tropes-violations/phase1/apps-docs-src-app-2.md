# Tropes Audit: apps/docs/src/app — Slice 2

**Files audited**:

- `src/app/analytics/page.tsx` (Analytics)
- `src/app/behavioral/page.tsx` (Behavioral Fingerprint)
- `src/app/clipboard/page.tsx` (Clipboard Intelligence)
- `src/app/contacts/page.tsx` (Contacts)
- `src/app/inbox/page.tsx` (Inbox)
- `src/app/keyboard-shortcuts/page.tsx` (Keyboard Shortcuts)
- `src/app/knowledge-sources/page.tsx` (Knowledge Sources)
- `src/app/meeting-prep/page.tsx` (Meeting Prep)

---

## src/app/analytics/page.tsx (Analytics)

**User-facing prose** (selected passages):

> "The Analytics page tracks your communication activity and the AI's performance over time. It shows how often AI drafts are accepted, how much time the AI has estimated saving you, and how your communication patterns break down across platforms. These numbers update as you use the product."

> "A time range selector at the top lets you view data by day, week, month, or year. Summary cards at the top show key numbers with percentage change from the previous period, so you can see trends at a glance."

> "Acceptance rate is the percentage of AI work items you accepted (sent or acted on), whether unedited or edited before sending. It is a key metric for measuring how useful the AI's suggestions are to your workflow."

> "When the AI generates a draft reply, you have three options: accept it as-is, edit it before sending, or discard it entirely."

> "First week: The AI is calibrating to your style. Expect frequent edits."
> "After 2 weeks: Drafts start matching your tone and preferences more consistently."
> "After 1 month: Routine messages should need minimal editing."

> "Analytics data builds over time. Here is what to expect as the AI learns your patterns."

> "Enough data exists for trends. Acceptance rate should show upward movement..."

> "Check Analytics at the end of your first week. Even early numbers show the AI is working and learning."

**Findings**:

1. **Fractal summary / signposted conclusion**: "Analytics data builds over time. Here is what to expect as the AI learns your patterns." — prefatory sentence that restates what the section heading already says. Cut it; open with the Day 0-1 card directly.

2. **Stakes inflation / false precision**: "Even early numbers show the AI is working and learning." — vague reassurance masquerading as useful information. The preceding content already establishes that AI drafts start appearing within days; this sentence adds nothing.

3. **"key metric"** (acceptance rate section): Minor. "Key metric" is a common but mild bit of jargon inflation. Not severe.

4. **Pedagogical voice**: "Here is what to expect as the AI learns your patterns." — "here is what to expect" is a textbook pedagogical opener. Cut.

**Severity**: Low.

---

## src/app/behavioral/page.tsx (Behavioral Fingerprint)

**User-facing prose** (selected passages):

> "The behavioral fingerprint is the AI's understanding of how you communicate. It tracks your tone by channel, your formality with different people, your response speed patterns, and your peak working hours. This fingerprint is what allows the AI to draft replies that sound like you, not like a generic assistant."

> "Unlike a simple writing style model, it captures the full context of how you work..."

> "The behavioral fingerprint is the foundation of all AI personalization. Improvements here propagate to every feature: better drafts, smarter triage, more relevant morning briefs."

**Findings**:

1. **Fractal summary / signposted conclusion**: "The behavioral fingerprint is the foundation of all AI personalization. Improvements here propagate to every feature: better drafts, smarter triage, more relevant morning briefs." — This closing callout is a recap of what the entire page just explained. It restates the fingerprint's role without adding new information. Classic fractal summary.

2. **"Unlike a simple writing style model"**: Negative parallelism. The copy defines what the fingerprint is by contrasting it with what it is not. This is a flagged pattern. The fingerprint should be described directly.

3. **"foundation of all AI personalization"**: Ornate noun phrasing. "Foundation" used metaphorically to elevate a feature description.

**Severity**: Low-medium. The callout box at the bottom of the page is the clearest instance; the negative parallelism sentence is also actionable.

**Specific violations**:

- Line ~66-68: "Unlike a simple writing style model, it captures the full context..." — rewrite to describe directly what it captures without the negative contrast.
- Closing callout (line ~228-234): Replace with a shorter pointer to Analytics, dropping the "foundation of all AI personalization" summary sentence.

---

## src/app/clipboard/page.tsx (Clipboard Intelligence)

**User-facing prose** (selected passages):

> "Clipboard Intelligence monitors your clipboard for workspace URLs and surfaces relevant context from your connected platforms."

> "This saves the context-switching overhead of opening the source app just to get details about a link someone shared with you."

> "Clipboard Intelligence runs entirely on your device. URL parsing happens locally. Only recognized workspace URLs trigger a context lookup against your synced data."

**Findings**: None. Prose is direct and precise. No flagged patterns.

---

## src/app/contacts/page.tsx (Contacts)

**User-facing prose** (selected passages):

> "SlopWeaver unifies identities across Gmail, Slack, Outlook, and Linear. AI links the same person across platforms, you confirm the connections."

> "SlopWeaver links identities. You confirm the connections. Nothing merges without you."

> "Entity resolution powers the AI's per-recipient style matching. When the AI knows that 'Sarah from engineering' is the same person on Gmail, Slack, and Linear, it can draft replies with the right tone for each platform. This contributes to higher acceptance rates in Analytics."

**Findings**:

1. **Fractal summary**: The closing callout — "Entity resolution powers the AI's per-recipient style matching... This contributes to higher acceptance rates in Analytics." — is a re-explanation of a concept the page just covered in full. The callout adds a forward link but the explanatory sentence is redundant.

**Severity**: Low.

---

## src/app/inbox/page.tsx (Inbox)

**User-facing prose** (selected passages):

> "SlopWeaver prioritizes messages based on your patterns. Priority scores adjust as you use the app. Messages from Gmail, Slack, Outlook, and Linear appear in one stream, ranked by what you actually care about."

> "SlopWeaver shows messages sorted by priority ordering, but you can filter, search, and organize however you want. Suggestions help. You decide."

> "Every message you read, reply to, or archive in the inbox feeds the AI's understanding of your priorities. Over time, priority ordering and insights become more accurate. Track the improvement in Analytics."

**Findings**:

1. **Fractal summary**: Closing callout — "Every message you read, reply to, or archive in the inbox feeds the AI's understanding of your priorities. Over time, priority ordering and insights become more accurate." — re-explains the learning loop after the page has already introduced it. The second sentence adds no new information.

**Severity**: Low.

---

## src/app/keyboard-shortcuts/page.tsx (Keyboard Shortcuts)

**User-facing prose** (selected passages):

> "SlopWeaver keeps it simple. One shortcut to rule them all."

> "Opens the command palette - your gateway to everything in SlopWeaver."

> "Open the command palette and start typing. No need to memorize commands - just describe what you want in plain English."

**Findings**:

1. **"your gateway to everything"**: Ornate noun phrase. "Gateway" is marketing language inflating what is a search bar. "Opens the command palette — search and run any action" would be direct.

2. **"One shortcut to rule them all"**: Pop-culture reference used as a hook. Not technically a listed trope, but it is a form of false levity that trades on an external reference rather than making a claim about the product. Minor.

**Severity**: Low.

---

## src/app/knowledge-sources/page.tsx (Knowledge Sources)

**User-facing prose** (selected passages):

> "Knowledge Sources let you upload documents, ChatGPT or Claude conversation exports, URLs, and plain text. The AI parses this content and uses it as context when drafting replies, extracting TODOs, and preparing meeting briefs. This accelerates the learning process by giving the AI your existing knowledge on day one, instead of waiting for it to discover everything from your messages."

> "Knowledge Sources accelerate the AI's learning by providing context that would otherwise take weeks to accumulate from synced messages alone."

> "Import your ChatGPT or Claude history on day one. This gives the AI a head start on understanding your interests, communication style, and the topics you work on, which directly improves early acceptance rates in Analytics."

**Findings**:

1. **Fractal summary**: "Knowledge Sources accelerate the AI's learning by providing context that would otherwise take weeks to accumulate from synced messages alone." — this is a near-verbatim restatement of the lead paragraph ("This accelerates the learning process by giving the AI your existing knowledge on day one, instead of waiting for it to discover everything from your messages"). The section "How It Helps the AI Learn" opens by re-explaining what was already said in the intro.

2. **Pedagogical voice**: "instead of waiting for it to discover everything from your messages" — slightly inflated framing, implying the product is rescuing the user from a painful status quo.

**Severity**: Low. The duplicate in the "How It Helps" section is the main actionable item.

---

## src/app/meeting-prep/page.tsx (Meeting Prep)

**User-facing prose** (selected passages):

> "Meeting Prep automatically assembles context for your upcoming calendar events. Before each meeting, the AI searches across all connected platforms for relevant messages, documents, and tasks related to the meeting topic and attendees."

> "This is not a generic summary. The briefing is personalized based on your behavioral fingerprint and your actual communication history with each attendee."

> "Meeting prep improves as the AI learns more about your work. The more platforms connected and the more messages synced, the richer the briefings become."

**Findings**:

1. **Negative parallelism**: "This is not a generic summary." — defines the feature by what it is not. The following sentence ("The briefing is personalized...") makes the positive claim; the negative opener is redundant and a flagged pattern.

2. **Fractal summary**: Closing callout — "The more platforms connected and the more messages synced, the richer the briefings become." — restates the general principle (more data = better AI) that appears throughout the docs. No new information specific to meeting prep.

**Severity**: Low.

**Specific violation**:

- Line ~65: "This is not a generic summary." — delete; keep only "The briefing is personalized based on your behavioral fingerprint and your actual communication history with each attendee."

---

## Summary for Slice 2

| File                        | Findings                                                                   |
| --------------------------- | -------------------------------------------------------------------------- |
| analytics/page.tsx          | 3 low findings (fractal summary, pedagogical opener, vague reassurance)    |
| behavioral/page.tsx         | 3 low-medium findings (fractal summary, negative parallelism, ornate noun) |
| clipboard/page.tsx          | None                                                                       |
| contacts/page.tsx           | 1 low finding (fractal summary in closing callout)                         |
| inbox/page.tsx              | 1 low finding (fractal summary in closing callout)                         |
| keyboard-shortcuts/page.tsx | 2 low findings (ornate noun, pop-culture hook)                             |
| knowledge-sources/page.tsx  | 2 low findings (fractal summary, pedagogical framing)                      |
| meeting-prep/page.tsx       | 2 low findings (negative parallelism, fractal summary)                     |

**Priority violations to fix**:

1. `behavioral/page.tsx`: "Unlike a simple writing style model" — remove negative parallelism; describe directly.
2. `behavioral/page.tsx` closing callout: Cut the "foundation of all AI personalization" recap sentence.
3. `meeting-prep/page.tsx` line ~65: Delete "This is not a generic summary."
4. `analytics/page.tsx`: Delete "Here is what to expect as the AI learns your patterns." and "Even early numbers show the AI is working and learning."
5. `keyboard-shortcuts/page.tsx`: Replace "your gateway to everything in SlopWeaver" with a direct description.
