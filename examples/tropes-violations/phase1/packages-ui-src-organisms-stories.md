# Slice 199: packages/ui/src/organisms/stories

Files audited:

- `packages/ui/src/organisms/stories/inbox-navigation-patterns.stories.tsx`

---

## inbox-navigation-patterns.stories.tsx

This is a Storybook-only file documenting layout and navigation patterns. However, several story `render()` blocks contain hardcoded strings that simulate product UI visible in Storybook previews. These need to be reviewed as representative copy.

**User-facing strings (in rendered stories):**

- `"AI Assistant"` — label on the right panel (line 211).
- `"Select a message to prepare a reply or search for context"` — placeholder text in AI panel (line 213-215).
- `"No integrations connected"` — empty state heading (line 403).
- `"Connect Slack, Gmail, or Linear to see your messages"` — empty state body (line 406-407).
- `"Connect Integrations"` — CTA button (line 410).
- `"Triage!"` — empty state heading (line 420).
- `"You're all caught up. Great job!"` — empty state body (line 424).
- `"No messages found"` — empty state heading (line 437).
- `"Try adjusting your filters or connecting more platforms"` — empty state body (line 441-442).
- `"Clear Filters"` — button (line 444).
- Priority label descriptions: `"Critical issues"`, `"Important tasks"`, `"Regular work"`, `"Nice to have"` (lines 478-513).

### Violation: "Select a message to prepare a reply or search for context"

**File:** `packages/ui/src/organisms/stories/inbox-navigation-patterns.stories.tsx`, lines 213-215

```tsx
<Text className="text-muted-foreground" size="sm">
  Select a message to prepare a reply or search for context
</Text>
```

**Trope:** "Search for context" is AI jargon leaking into UI copy. "Context" is a technical AI term. Users don't search for "context" — they search for information, related emails, or background. This reads like a developer-authored placeholder that was never rewritten for users.

**Severity:** Low-medium. This text represents what would appear in the AI assistant panel's empty state, which is a high-visibility area when the user hasn't selected anything.

### Violation: "You're all caught up. Great job!"

**File:** `packages/ui/src/organisms/stories/inbox-navigation-patterns.stories.tsx`, line 424

```tsx
<Text className="text-muted-foreground" size="sm">
  You're all caught up. Great job!
</Text>
```

**Trope:** Hollow praise / AI cheerleading. "Great job!" is the textbook example of AI apps congratulating users for doing nothing unusual. Inbox zero is an achievement but "Great job!" is condescending — the product is praising the user for using the product. This pattern is mocked widely as a UX anti-pattern.

**Severity:** Medium. The empty state is shown whenever the inbox is cleared, making this copy repeat frequently.

**Recommendation:** Replace with something that respects the user: "Inbox clear." or "All done for now." No exclamation point, no praise.

### Violation: "Triage!"

**File:** `packages/ui/src/organisms/stories/inbox-navigation-patterns.stories.tsx`, line 420

```tsx
<Text size="lg" weight="semibold">
  Triage!
</Text>
```

**Trope:** The heading "Triage!" is internal product jargon used as a celebratory empty state label. This sounds like a developer-chosen internal name for the state, not a message to the user. Combined with "Great job!" beneath it, the empty state double-dips on hollow praise.

**Severity:** Low. Heading text on an empty state.

### Note on stories vs. product copy

This file is a Storybook story file (`*.stories.tsx`). The strings above appear in rendered story previews, not shipped product code. However, they represent the copy direction for these UI states and are worth flagging as patterns to avoid when the real components are built.

---

## Findings

| File                                    | Line    | String                                                        | Trope                                  | Severity   |
| --------------------------------------- | ------- | ------------------------------------------------------------- | -------------------------------------- | ---------- |
| `inbox-navigation-patterns.stories.tsx` | 213-215 | `"Select a message to prepare a reply or search for context"` | "Context" is AI jargon in user copy    | Low-medium |
| `inbox-navigation-patterns.stories.tsx` | 420     | `"Triage!"`                                                   | Internal jargon as empty-state heading | Low        |
| `inbox-navigation-patterns.stories.tsx` | 424     | `"You're all caught up. Great job!"`                          | Hollow AI praise / cheerleading        | Medium     |
