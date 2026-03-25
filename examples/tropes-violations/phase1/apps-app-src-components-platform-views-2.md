# Tropes Audit: apps/app/src/components/platform-views (Slice 90)

Files audited:

- github-view.tsx
- google-calendar-event-view.tsx
- google-gmail-thread-view.tsx
- inline-reply-composer.tsx
- issue-task-view.tsx
- jira-issue-view.tsx
- linear-issue-view.tsx
- linear-priority-icon.tsx

---

## Findings

### 1. `inline-reply-composer.tsx` — button label "Generate suggestion"

**File**: `apps/app/src/components/platform-views/inline-reply-composer.tsx`
**Line**: 84
**Text**: `"Generate suggestion"`

**Why it's a trope**: Hedge-word AI filler. "Suggestion" frames the AI output as tentative and deferential. Every AI startup in 2023-2024 used this exact phrasing. It says nothing about what the AI actually does (draft a reply in your voice, continue the thread).

**Fix direction**: Use action-oriented copy that describes the outcome. Options: "Draft reply", "Write reply", "AI draft".

---

### 2. `inline-reply-composer.tsx` — loading label "Generating..."

**File**: `apps/app/src/components/platform-views/inline-reply-composer.tsx`
**Line**: 84
**Text**: `"Generating..."`

**Why it's a trope**: Generic AI loading state that appears in every LLM product. No specificity about what is being generated.

**Fix direction**: "Drafting...", "Writing...", or "Working..." — anything that names the action.

---

### 3. `google-gmail-thread-view.tsx` — placeholder "Write your reply..."

**File**: `apps/app/src/components/platform-views/google-gmail-thread-view.tsx`
**Line**: 623
**Text**: `placeholder="Write your reply..."`

**Why it's a trope**: Inert filler placeholder. Every email client uses this. It occupies the space where a meaningful prompt or nudge could live.

**Fix direction**: Something that implies the AI context, e.g. "Add to the draft or type your own reply" — but only if that accurately describes the UX state. If the composer is purely manual, a blank placeholder or none at all is preferable to this generic line.

---

### 4. `github-view.tsx` — placeholder "Leave a comment... (Cmd+Enter to submit)"

**File**: `apps/app/src/components/platform-views/github-view.tsx`
**Line**: 359
**Text**: `placeholder="Leave a comment... (Cmd+Enter to submit)"`

**Why it's a trope**: "Leave a comment" is the stock GitHub placeholder text. It adds no SlopWeaver identity. Minor — the keyboard shortcut hint is useful and should be retained regardless.

**Fix direction**: Not urgent; keyboard hint redeems it. If changed, preserve the shortcut hint.

---

### 5. `google-calendar-event-view.tsx` — empty-state copy with passive "No X provided/listed"

**File**: `apps/app/src/components/platform-views/google-calendar-event-view.tsx`
**Lines**: 42, 60, 86, 101
**Text examples**:

- `"No event description."`
- `"No location provided."`
- `"No meeting link."`
- `"No attendees listed."`

**Why it's a trope**: Filler empty states. These are technically functional but contribute to a flat, corporate-tool feel. Each one is a missed micro-moment. Not urgent — many are genuinely appropriate for a data-driven view — but the pattern is worth noting.

**Fix direction**: Low priority. Acceptable as-is for calendar event data. If ever refreshed, consider context-aware cues ("No location — might be remote") but only where it adds real value.

---

### 6. `jira-issue-view.tsx` — toast copy "Copy not available in this context"

**File**: `apps/app/src/components/platform-views/jira-issue-view.tsx`
**Line**: 95
**Text**: `"Copy not available in this context"`

**Why it's a trope**: Vague error message. "In this context" is a placeholder phrase developers write when they don't know what to say. Users cannot act on it.

**Fix direction**: "Couldn't copy — check clipboard permissions" or simply "Copy failed".

---

### 7. `linear-issue-view.tsx` — same "Copy not available in this context" pattern

**File**: `apps/app/src/components/platform-views/linear-issue-view.tsx`
**Line**: 141
**Text**: `"Copy not available in this context"`

Same issue as jira-issue-view.tsx above.

---

## Summary

| #   | File                           | Line(s)      | Offending text                       | Category                   |
| --- | ------------------------------ | ------------ | ------------------------------------ | -------------------------- |
| 1   | inline-reply-composer.tsx      | 84           | "Generate suggestion"                | AI hedge-word trope        |
| 2   | inline-reply-composer.tsx      | 84           | "Generating..."                      | Generic AI loading state   |
| 3   | google-gmail-thread-view.tsx   | 623          | "Write your reply..."                | Inert stock placeholder    |
| 4   | github-view.tsx                | 359          | "Leave a comment..."                 | Stock platform placeholder |
| 5   | google-calendar-event-view.tsx | 42,60,86,101 | "No X provided/listed."              | Passive empty-state filler |
| 6   | jira-issue-view.tsx            | 95           | "Copy not available in this context" | Vague error copy           |
| 7   | linear-issue-view.tsx          | 141          | "Copy not available in this context" | Vague error copy           |

Priority order: 1 and 2 are highest-impact because `InlineReplyComposer` is shared across all platform views — fixing those two strings propagates to every platform.
