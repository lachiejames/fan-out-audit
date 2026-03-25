# Phase 2: Cross-Cutting Patterns (Group 17)

**Slices analyzed**: 10 (packages/ui/src/organisms -- Calendar, Search, Workflows, chat slices 1-2, inbox slices 1-2, stories, table, top-level organisms)

---

## Pattern 1: "AI-generated reply" repeated across all platform views

**Slices affected**: inbox slice 2 (GmailThreadView, LinearIssueView, SlackThreadView) -- 3 files, 1 slice

The string `"AI-generated reply"` appears identically in all three platform-specific composer views:

- `GmailThreadView.tsx` line 482
- `LinearIssueView.tsx` line 356
- `SlackThreadView.tsx` line 214

This is a system-internal provenance label ("this was made by AI") rather than a user-benefit frame. It tells the user what made the content, not what they should do with it. The repetition across three files means any copy improvement needs to land in three places simultaneously.

**Suggested fix**: Replace with an action-oriented label like "Review before sending" or a branded label like "SlopWeaver draft". Alternatively, let the Regenerate/Discard buttons imply AI authorship without a separate label.

---

## Pattern 2: Hedged AI output labels that avoid committing to what the AI did

**Slices affected**: inbox slices 1-2 (MessageActionBar, MessageCard) -- 2 files across 2 slices

Multiple labels in the inbox use deliberately soft language to describe AI actions:

| File                 | String                                     | Issue                                                                     |
| -------------------- | ------------------------------------------ | ------------------------------------------------------------------------- |
| MessageActionBar.tsx | `"Prepare reply"` / `"Preparing reply..."` | "Prepare" is vague -- is the AI writing it or organizing background info? |
| MessageCard.tsx      | `"Reply suggestion"`                       | Soft label that avoids "AI draft" or "Draft reply"                        |
| MessageCard.tsx      | `"AI suggested"` (badge)                   | Says the AI did something, but not what or why                            |

These labels share a common trait: they hedge on the AI's actual role. The system is generating a draft reply, but the copy frames it as "preparing" or "suggesting" rather than "drafting" or "writing". This hedging pattern makes the product feel uncertain about its own capabilities.

**Suggested fix**: Be direct about what the AI did. "Draft reply" instead of "Prepare reply". "AI draft" or "Your draft" instead of "Reply suggestion". Replace "AI suggested" badge with an action cue like "Reply ready" or "Needs reply".

---

## Pattern 3: Vague "confidence" labels leaking AI internals to users

**Slices affected**: inbox slice 1 (message-card-helpers.ts), also related to ConfidenceBar in Group 16 (cross-group pattern)

The confidence label system in `message-card-helpers.ts` maps numeric scores to vague tiers:

```
>= 80 -> "High confidence"
>= 60 -> "Medium confidence"
< 60  -> "Low confidence"
```

"Confidence" is AI-system vocabulary. Users don't know what the confidence measures -- confidence that the reply matches their tone? That it is factually correct? That it will be accepted? This pattern also appeared in Group 16's ConfidenceBar component (`"AI confidence: X%"` aria-label), confirming it is a cross-cutting issue in how the product surfaces AI certainty to users.

**Suggested fix**: Either replace with action-oriented language ("Looks ready to send" / "Worth a review" / "Needs your edit") or remove the label entirely, showing the score only in a developer-facing tooltip.

---

## Pattern 4: "Context" as AI jargon in user-facing copy

**Slices affected**: organisms/stories (inbox-navigation-patterns), also related to ContextBar in Group 16 (cross-group pattern)

The inbox navigation stories contain `"Select a message to prepare a reply or search for context"`. The word "context" appears as a thing users should "search for", which is AI-system framing. Users search for information, related messages, or background, not "context".

Combined with the ContextBar component from Group 16 (which uses `"Context"` as a section header), this confirms a cross-cutting pattern: the product treats "context" as a user-facing concept when it is actually an internal AI-system term.

**Suggested fix**: Replace "context" with concrete descriptions of what the user is actually looking for or looking at.

---

## Pattern 5: Hollow praise in empty states

**Slices affected**: organisms/stories (inbox-navigation-patterns) -- 1 slice

The inbox empty state uses `"Triage!"` as a heading with `"You're all caught up. Great job!"` as body text. "Great job!" is a widely mocked AI-product anti-pattern -- the product congratulating users for a routine state (inbox zero). The heading "Triage!" is internal product jargon used as celebratory copy.

Note: this appeared in a Storybook story file, not shipped product code. However, it represents the copy direction for the inbox empty state and seeds the pattern for when the real component is built.

**Suggested fix**: Replace with neutral, respectful copy. "All done for now." or "Inbox clear." No exclamation points, no praise.

---

## Pattern 6: Vague "AI has context" capability claim on meeting cards

**Slices affected**: Calendar (MeetingCard) -- 1 slice, also thematically related to ContextBar and inbox stories (cross-group)

The MeetingCard component displays `"AI has context"` as an inline label when the AI has background information for a meeting. This is the same abstract "context" framing seen in the ContextBar (Group 16) and inbox stories (this group). "Context" is undefined from the user's perspective -- it could mean related emails, attendee info, or meeting notes.

**Suggested fix**: Replace with what the AI actually has: "Has related emails", "3 related messages", or the count/type of background material.

---

## Pattern 7: Chat and table organisms are clean

**Slices affected**: chat slices 1-2, Search, Workflows, table, top-level organisms (7 of 10)

Seven of ten slices had zero findings. The chat components (code-block, controls, response, shimmer, tool-content, tool-header, tool), search result items, workflow nodes, table infrastructure, and top-level organisms (table.tsx, unified-logo-mark) contain either no user-visible text or plain functional labels (status badges like "Completed", "Running", "Error"; action labels like "Edit", "Delete").

---

## Summary

11 violations across 3 slices (Calendar, inbox slices 1-2), plus 3 violations in story files that represent copy direction. The dominant cross-cutting patterns are: (1) the "AI-generated reply" label repeated across three platform views, (2) hedged language that avoids stating what the AI actually did, and (3) "context" and "confidence" as AI-internal vocabulary leaking into user-facing copy. Patterns 3, 4, and 6 connect to findings in Group 16 (ConfidenceBar, ContextBar), confirming these are systemic across the UI package's AI-feature components.
