# Tropes Audit: apps/app/src/components/inbox (Slice 83)

Files audited:

- `PredictionBadge.tsx`
- `PredictionPanel.tsx`
- `bulk-action-bar.tsx`
- `inbox-ai-suggestion-chips.tsx`
- `inbox-filter-bar.tsx`
- `inbox-filter-dropdown.tsx`
- `inbox-filter-pill.tsx`
- `message-card.tsx`

---

## PredictionBadge.tsx

**File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/components/inbox/PredictionBadge.tsx`

### Violations

**1. Tooltip: vague AI capability claim without proof**

- Location: line 36, tooltip text `"AI suggested a reply based on your writing style"`
- The phrase "based on your writing style" is an unsubstantiated claim in UI copy. It asserts the mechanism of learning without showing any metric. Per the Content Playbook, in-app copy must show real numbers from the user's data, not claim adaptation. The claim is not backed by any confidence number, pattern count, or other measurable signal shown to the user.
- Fix: Replace with something that describes what happened, e.g. `"Draft ready. Review before sending."` or, if the confidence score is surfaced, `"Reply drafted at ${confidence}% confidence."` The writing style claim belongs on the analytics page where it can be substantiated.

**2. Label "Reply ready" duplicated in message-card.tsx**

- This is a cross-file consistency note rather than a trope per se, but the badge label `"Reply ready"` (line 34) and the inline span in `message-card.tsx` (line 241, `"Reply ready"`) are two separate rendering paths for what appears to be the same concept. One is a button with a click handler (prediction not yet viewed), the other is a read-only badge (AI work item already generated). The distinction is not surfaced to the user; both say the same thing. This risks user confusion and is noted for UX review.

---

## PredictionPanel.tsx

**File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/components/inbox/PredictionPanel.tsx`

### Violations

**1. Info banner: passive vague claim**

- Location: line 52-54, `"This response was generated from your context. Edit or send when ready."`
- "Generated from your context" is meaningless to the user. It names no specific source (which emails, which writing patterns, which integrations). Per the Content Playbook, in-app copy should show what's actually happening with the user's real data.
- Fix: Either drop the banner entirely (the header "Reply suggestion" with a review note is sufficient) or replace with something concrete, e.g. `"Drafted using your last 3 replies to this sender."` if that data is available.

**2. Header subtext uses em dash**

- Location: line 44, `"— Review before sending"`
- Em dashes are banned everywhere per the Content Playbook formatting rules.
- Fix: `"Review before sending"` as a separate sentence, or use a comma: `"Reply suggestion, review before sending"`.

---

## bulk-action-bar.tsx

No trope violations found. Copy is functional and direct ("Archive", "Mark as read", "Mark as unread", "Delete", "Select all", "Clear selection"). No AI claims, no banned phrases.

---

## inbox-ai-suggestion-chips.tsx

**File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/components/inbox/inbox-ai-suggestion-chips.tsx`

### Violations

**1. JSDoc comment uses "brand gradient" framing**

- Location: line 94, `"Renders primary suggestion with brand gradient"`
- Minor: not user-visible, but the JSDoc describes a visual treatment ("brand gradient") that is associated with the "AI slop aesthetic" the design system explicitly bans (gradient buttons banned in design-system.md). This is a code comment, not UI copy, so it is low priority, but it reflects a framing that may influence future UI decisions.

No user-visible trope violations in this file. The component delegates all visible text to `SuggestionChips`.

---

## inbox-filter-bar.tsx

No trope violations found. All visible copy is functional UI labels: "All Platforms", "Inbox", "All mail", "Archived", "Newest first", "Oldest first", "Highest priority", "Lowest priority", "All", "Priority", "Active:", "Clear all", "Save filter", "Filter name...", "Save", "Cancel". No AI claims, no banned phrases.

---

## inbox-filter-dropdown.tsx

No trope violations found. The component renders option labels passed in by the parent with no hardcoded copy of its own.

---

## inbox-filter-pill.tsx

No trope violations found. No copy of its own; renders the `label` prop passed by the parent.

---

## message-card.tsx

**File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/components/inbox/message-card.tsx`

### Violations

**1. Hardcoded badge "Reply ready" as read-only span**

- Location: lines 238-242 (desktop) and lines 327-331 (mobile)
- The badge `"Reply ready"` is shown when `message.hasAIWorkItem` is true. This is a static claim that something is ready without showing any supporting metric (confidence score, draft preview, which platform generated it). Per the Content Playbook, in-app copy should show real data, not assertions.
- Additionally, on mobile (line 329) the badge is shortened to just `"Reply"`, which loses even the minimal meaning of the desktop label.
- Fix: Either link this badge to the prediction panel (which is already accessible via `PredictionBadge`) so it functions as an entry point rather than a passive assertion, or replace the label with something that names the action: `"Draft ready"` with a tap target that opens the draft. The mobile truncation to `"Reply"` should at minimum say `"Draft"` to distinguish it from the manual reply action.

**2. "VIP" badge lacks explanation**

- Location: lines 202-207
- The `"VIP"` badge appears on the message card but the user is given no in-context explanation of why a sender is VIP or how to change it. This is a UX gap rather than a trope violation per se, but the label is an assertion without proof ("this sender is important") which runs counter to the Content Playbook's principle of showing evidence, not claims. Noted for UX review.

---

## Summary

| File                          | Violations                                                                    |
| ----------------------------- | ----------------------------------------------------------------------------- |
| PredictionBadge.tsx           | 1 substantive (vague AI claim in tooltip), 1 UX note                          |
| PredictionPanel.tsx           | 1 substantive (vague "generated from your context"), 1 banned em dash         |
| bulk-action-bar.tsx           | None                                                                          |
| inbox-ai-suggestion-chips.tsx | 1 minor (JSDoc only, not user-visible)                                        |
| inbox-filter-bar.tsx          | None                                                                          |
| inbox-filter-dropdown.tsx     | None                                                                          |
| inbox-filter-pill.tsx         | None                                                                          |
| message-card.tsx              | 1 substantive (static "Reply ready" badge without proof or action), 1 UX note |

**Priority fixes:**

1. `PredictionPanel.tsx` line 44: remove em dash (banned, trivial fix)
2. `PredictionPanel.tsx` line 52: replace "generated from your context" with concrete source or remove
3. `PredictionBadge.tsx` line 36: replace "based on your writing style" with something measurable or action-oriented
4. `message-card.tsx` lines 238-242, 327-331: replace passive "Reply ready" badge with an action entry point or substantiated claim
