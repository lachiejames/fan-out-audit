# Slice 197: packages/ui/src/organisms/inbox (8 files)

Files audited:

- `packages/ui/src/organisms/inbox/AIResponseBubble.tsx`
- `packages/ui/src/organisms/inbox/CalendarStrip.tsx`
- `packages/ui/src/organisms/inbox/EmailIframeViewer.tsx`
- `packages/ui/src/organisms/inbox/InboxFilters.tsx`
- `packages/ui/src/organisms/inbox/MessageActionBar.tsx`
- `packages/ui/src/organisms/inbox/message-action-bar-helpers.ts`
- `packages/ui/src/organisms/inbox/message-card-helpers.ts`
- `packages/ui/src/organisms/inbox/InboxFilters.stories.tsx`

---

## AIResponseBubble.tsx

**User-facing strings:**

- `"Summary"` — section heading (line 45).
- `"Found {highlightCount} messages"` — badge text (line 47).
- `"View All"` — action button (line 60).

No tropes. These are functional labels showing factual output counts.

---

## CalendarStrip.tsx

**User-facing strings:**

- `"No events today"` (line 95) — shown when event list is empty.
- `"Hide calendar"` (line 197) — collapse button.
- `"Today's Events"` (line 145) — expanded panel heading.
- `"{n} event(s) scheduled"` (line 147).

No violations. All functional, factual labels.

---

## EmailIframeViewer.tsx

**User-facing strings:**

- `"Scroll to see more →"` (line 177) — overflow indicator.
- `"Email content"` — iframe `title` attribute (line 199).

Functional strings. No violations.

---

## InboxFilters.tsx

**User-facing strings (selected):**

- Smart view labels: `"All Messages"`, `"Needs Reply"`, `"Important"`, `"From Team"`, `"Urgent"`
- Smart view descriptions: `"Everything in your inbox"`, `"Unread messages requiring attention"`, `"Starred or priority messages"`, `"Messages from your team"`, `"High priority items"`
- `"Smart Views & Filters"` — panel header (line 327)
- `"Customize your inbox view"` — panel subheading (line 333)
- `"Smart Views"`, `"Quick Toggles"`, `"Platforms"`, `"Time Range"`, `"Sort By"` — section labels
- `"Unread only"`, `"Starred/Important"` — toggle labels
- Sort descriptions: `"Newest messages first"`, `"High priority messages first"`, `"Unread messages first"`
- `"Advanced Conditions"` — section label
- `"Condition {n}"` — condition numbering
- `"Enter value..."` — input placeholder
- `"Add Condition"` — button
- `"Saved Filters"` — section label
- `"Filter name..."` — input placeholder
- `"Save Current Filter"` — button
- `"Apply Filters"`, `"Reset All"` — footer buttons
- `"Edit Filter"` / `"Save Filter"` — save dialog heading

### Violation: "Customize your inbox view"

**File:** `packages/ui/src/organisms/inbox/InboxFilters.tsx`, line 333

```tsx
<p className="text-muted-foreground text-xs">Customize your inbox view</p>
```

**Trope:** Generic filler subheading. "Customize your inbox view" states the obvious — the panel is literally named "Smart Views & Filters". This is unnecessary copy that adds no information. Classic AI-generated padding.

**Severity:** Low. Small string in a secondary position, but it's the kind of meaningless subtext that accumulates to produce a "corporate AI startup" tone.

---

## MessageActionBar.tsx

**User-facing strings:**

- `"Preparing reply..."` / `"Prepare reply"` — title tooltip on AI reply button (line 131).
- `"Ask about this message"` — tooltip on ask-AI button (line 146).
- `"Archive"`, `"Unstar"` / `"Star"`, `"More actions"` — icon button titles.
- `"{platformLabel} Actions"` — bottom sheet header (line 203).
- `"Mark Unread"` / `"Mark Read"`, `"Delete"`, `"Set Priority"`, `"Assign"`, `"Add Label"`, `"Add Reaction"`, `"Change Status"` — action labels.

### Violation: "Prepare reply"

**File:** `packages/ui/src/organisms/inbox/MessageActionBar.tsx`, line 131

```tsx
title={isGeneratingReply ? "Preparing reply..." : "Prepare reply"}
```

**Trope:** Softened AI framing. "Prepare reply" is deliberately vague about whether the AI is writing the reply or the user is composing one. Compare to clearer alternatives: "Draft reply", "Write reply", or "Generate reply". The hedged word "prepare" is typical of AI copy that avoids committing to what the AI actually does.

**Severity:** Low. Tooltip text only, not a headline.

---

## message-action-bar-helpers.ts

**User-facing strings:**

- `"Reply in Thread"` — messaging platforms.
- `"Add Comment"` — project management platforms.
- `"Reply"` — fallback.

Functional, specific. No violations.

---

## message-card-helpers.ts

**User-facing strings:**

- `"High confidence"`, `"Medium confidence"`, `"Low confidence"` — confidence labels for AI reply suggestions.

### Violation: Confidence labels

**File:** `packages/ui/src/organisms/inbox/message-card-helpers.ts`, lines 56-61

```typescript
if (confidence >= 80) return "High confidence";
if (confidence >= 60) return "Medium confidence";
return "Low confidence";
```

**Trope:** Vague AI confidence language. "High confidence" / "Medium confidence" / "Low confidence" are meaningless to users without knowing what the confidence measure represents or what they should do differently based on it. This is AI-system-internal vocabulary leaking into the UI. Users don't know what "confidence" means here — confidence in what? That the reply is accurate? That it matches their tone? That it will be accepted?

**Severity:** Medium. These labels appear in the MessageCard reply suggestion area and directly influence how users perceive AI quality. Vague confidence framing is a known AI writing anti-pattern.

**Recommendation:** Replace with action-oriented cues like "Looks ready to send", "Worth a quick review", or "Needs your edit" — or remove the label entirely and show the score only in a tooltip for developers.

---

## InboxFilters.stories.tsx

Storybook-only file. No user-facing product copy. No violations.

---

## Findings

| File                      | Line  | String                                                           | Trope                                                   | Severity |
| ------------------------- | ----- | ---------------------------------------------------------------- | ------------------------------------------------------- | -------- |
| `InboxFilters.tsx`        | 333   | `"Customize your inbox view"`                                    | Filler subheading, states the obvious                   | Low      |
| `MessageActionBar.tsx`    | 131   | `"Prepare reply"`                                                | Hedged AI framing, vague about AI authorship            | Low      |
| `message-card-helpers.ts` | 56-61 | `"High confidence"` / `"Medium confidence"` / `"Low confidence"` | Internal AI vocabulary leaking to UI, undefined meaning | Medium   |
