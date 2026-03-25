# Tropes Audit: apps/docs/src/app — Slice 1

**Files audited**:

- `src/app/page.tsx` (docs hub / homepage)
- `src/app/layout.tsx` (root layout)
- `src/app/not-found.tsx` (404 page)
- `src/app/global-error.tsx` (error boundary)
- `src/app/posthog-provider.tsx` (analytics provider)
- `src/app/quick-start/page.tsx` (Quick Start guide)
- `src/app/ai/page.tsx` (AI Chat page)
- `src/app/ai-limits/page.tsx` (Actions & Limits page)

---

## src/app/page.tsx (Docs Hub)

**User-facing prose**:

> "SlopWeaver connects your work tools and learns your patterns."

> "The AI observes how you communicate across 17 platforms, builds a behavioral fingerprint of your work style, and uses that understanding to draft replies, triage messages, and prepare you for meetings. The Analytics page tracks how these improve over time."

> "Can't find what you're looking for? Support is available by email."

**Findings**: None. Prose is factual, direct, and free of AI writing tropes. No magic adverbs, no ornate nouns, no pedagogical voice, no false suspense, no stakes inflation.

---

## src/app/layout.tsx

**User-facing prose** (metadata only):

> "Learn how SlopWeaver's AI learns your patterns and improves over time."

**Findings**: None. Single metadata description sentence. Clean.

---

## src/app/not-found.tsx

**User-facing prose**:

> "This documentation page doesn't exist. It may have been moved or removed."

**Findings**: None.

---

## src/app/global-error.tsx

**User-facing prose**: None (only renders a Next.js error component).

**Findings**: None.

---

## src/app/posthog-provider.tsx

**User-facing prose**: None (analytics infrastructure, internal comments only).

**Findings**: None.

---

## src/app/quick-start/page.tsx (Quick Start)

**User-facing prose** (selected passages):

> "Get started with SlopWeaver quickly. Follow this guide to set up your account and connect your first integration."

> "You can skip manual setup by connecting your Gmail or Slack account. SlopWeaver works with whatever you connect."

> "SlopWeaver will now sync your messages and organize threads. You'll see live progress updates like: Threads imported / Action items found / Replies ready for review / Messages grouped by project"

> "Replies improve as you review and approve more messages."

> "Once syncing is complete, you'll land in your inbox with messages from all connected platforms. The AI starts learning your patterns immediately."

> "By the end of your first week, check Analytics to see your acceptance rate and estimated time saved. These metrics show the AI learning your patterns in real time."

> "Now that you're set up, explore these features:"

> "Need help getting started? Reach out for help."

**Findings**:

1. **Pedagogical voice / fractal summary**: "Get started with SlopWeaver quickly. Follow this guide to set up your account and connect your first integration." — "Get started quickly" is filler preamble telling the reader to do what the page already does. The lead of the page should just say what you'll accomplish, not announce that it's going to help you.

2. **Bold-first bullets with no payload**: The list "Threads imported / Action items found / Replies ready for review / Messages grouped by project" uses bullet-point phrasing that reads as illustrative fluff. These would be stronger as actual UI labels the user will see.

3. **Signposted conclusion**: "Now that you're set up, explore these features:" — low-value transitional sentence that could be cut entirely; the list speaks for itself.

4. **"in real time"** (line 173): Minor magic-adverb cluster. Not severe.

**Severity**: Low. The page is mostly clean. Items 1 and 3 are minor template-filler patterns.

---

## src/app/ai/page.tsx (AI Chat)

**User-facing prose**:

> "SlopWeaver's AI uses your synced messages, knowledge sources, and behavioral fingerprint to provide context-aware assistance. Reply suggestions are generated in the Queue. The AI improves over time as it learns your patterns, which you can track in Analytics."

> "Review AI-generated replies in the Queue. Adjust reply preferences and sync scope in Settings. Nothing sends without your approval."

**Findings**: None. Direct, factual, no flagged patterns.

---

## src/app/ai-limits/page.tsx (Actions & Limits)

**User-facing prose**:

> "Actions cover reply and analysis work. Reading and searching synced content is not metered."

> "Usage only changes when a reply or analysis runs. Browsing and search stay available even when usage is paused."

**Findings**: None. Terse and precise. The section heading "Usage stays predictable" is marketing-adjacent but not a trope per the audit checklist.

---

## Summary for Slice 1

| File                 | Findings                |
| -------------------- | ----------------------- |
| page.tsx (hub)       | None                    |
| layout.tsx           | None                    |
| not-found.tsx        | None                    |
| global-error.tsx     | None                    |
| posthog-provider.tsx | None                    |
| quick-start/page.tsx | 3 low-severity findings |
| ai/page.tsx          | None                    |
| ai-limits/page.tsx   | None                    |

**Violations requiring fix**:

- `quick-start/page.tsx` line ~43: "Get started with SlopWeaver quickly. Follow this guide to set up your account and connect your first integration." — remove the preamble and open with the first concrete step or a short statement of outcome.
- `quick-start/page.tsx` line ~181: "Now that you're set up, explore these features:" — cut this sentence; let the heading "Next Steps" carry the section.
