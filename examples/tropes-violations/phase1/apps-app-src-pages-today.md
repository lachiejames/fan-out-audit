# Trope Audit: apps/app/src/pages/today

Files audited:

- `meeting-prep-section.tsx`
- `morning-brief-panel.tsx`
- `page.tsx`
- `today-empty-state.tsx`
- `today-skeleton.tsx` (file does not exist at this path)

## Findings

### today-empty-state.tsx

**"Connect your tools first"** heading (line 62)
Category: Borderline. "Tools" is acceptable product language here, not a trope. No issue.

**"SlopWeaver will build your first morning brief after syncing."** (line 64)
Category: Passive AI agency framing. Minor. The sentence positions SlopWeaver as a subject that "builds" things on behalf of the user, which is fine brand voice, but "build" is a slightly inflated verb for what is essentially a report/digest. Not a serious violation.

**"Syncing your tools"** heading (line 83) / **"SlopWeaver is syncing your connected tools."** (line 85)
No issue. Literal description of the ongoing process.

**"Your first morning brief will be available once the initial sync completes."** (line 86)
Category: Minor passive phrasing. Not a trope. Acceptable.

**"Today's brief hasn't been generated yet"** heading (line 97)
No issue. Direct statement of state.

**"Your morning brief will be generated automatically based on your scheduled time, or you can generate one now."** (lines 99-100)
Category: Minor passive voice. "Generated automatically" is fine but "based on your scheduled time" is vague -- what scheduled time? If a schedule is not yet configured for the user, this sentence promises something that may not exist. This is a copy accuracy issue more than a trope. Flag for review.

### page.tsx

**"Loading your day..."** (line 149, loading state copy)
Category: AI-flavored anthropomorphism / cliche loading copy.
"Loading your day" is a common AI product trope -- treating a data fetch as something personal and experiential. Prefer a direct description: "Loading..." or "Fetching today's brief..." depending on which query is slowest.

**"Morning Brief"** section heading (line 196) / **"Meeting Prep"** (line 223) / **"Waiting for You"** (line 239)
These are product feature names, not tropes. "Waiting for You" slightly anthropomorphizes (threads don't wait) but it is an established section label elsewhere in the codebase, so it is consistent -- not a fresh violation.

**"Today's brief hasn't been generated yet."** (line 207)
No issue. Matches the empty state copy and is direct.

**"Generate now"** CTA (line 210)
No issue. Direct action label.

**"Brief History"** sidebar heading (line 256)
No issue.

**"Previous briefs will appear here."** (line 274)
Minor filler. "No previous briefs." would be tighter.

### morning-brief-panel.tsx

**"Needs Your Attention"** section title (line 113)
Category: Mild urgency framing. This is an established product section name; not a fresh trope introduced here. Consistent with the backend schema (`needsAttention`). No change needed.

**"Waiting for You"** section title (line 124)
Same assessment as above -- consistent product terminology.

**"Today's Focus"** section title (line 131)
No issue.

**"Regenerate"** button label (line 98)
No issue. Direct.

**"Scheduled"** badge label (line 89)
No issue.

**"Related items:"** label (line 189) / **"View #1", "View #2"** link labels (line 197)
Category: Vague link labels. "View #1" / "View #2" are machine-generated labels that convey no meaning to the user. Should be replaced with the item title or a descriptive label. Not a writing trope per se, but a UX copy failure that is common in AI-generated UI code.

### meeting-prep-section.tsx

**"Untitled Meeting"** fallback (line 63)
No issue. Defensive fallback label.

**"Attendees"** heading (line 90), **"Related"** heading (line 128)
"Related" is vague for a section heading. "Related emails" or "Related documents" would be more specific, but the content label (`content.source`) is shown alongside each item, so it is tolerable in context.

**"Join"** CTA (line 74)
No issue. Direct action.

### today-skeleton.tsx

File not found at the audited path. Nothing to audit.

## Summary

| File                     | Violations                                     |
| ------------------------ | ---------------------------------------------- |
| meeting-prep-section.tsx | 1 minor ("Related" heading vagueness)          |
| morning-brief-panel.tsx  | 1 minor ("View #1" / "View #2" link labels)    |
| page.tsx                 | 1 moderate ("Loading your day..." trope)       |
| today-empty-state.tsx    | 1 minor (vague "based on your scheduled time") |
| today-skeleton.tsx       | N/A (missing)                                  |

Total: 4 findings. The most actionable is "Loading your day..." in page.tsx -- a clear AI product cliche that should be replaced with literal copy. The "View #1" link labels in morning-brief-panel.tsx are a UX copy failure worth fixing.
