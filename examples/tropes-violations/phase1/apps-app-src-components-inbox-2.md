# Tropes Audit: apps/app/src/components/inbox (Slice 84)

Files audited:

- `mobile-filter-sheet.tsx`
- `suggestion-chips.tsx`

---

## mobile-filter-sheet.tsx

**File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/components/inbox/mobile-filter-sheet.tsx`

No trope violations found. All copy is functional UI labels: "Filters", "Clear all", "Inbox", "Archived", "All", "Priority", "Newest first", "Oldest first", "Highest priority", "Lowest priority", "Apply Filters", "Unknown integration". No AI claims, no banned phrases, no em dashes, no vague capability assertions.

---

## suggestion-chips.tsx

**File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/components/inbox/suggestion-chips.tsx`

### Violations

**1. Section label "Suggested actions" is a vague AI header**

- Location: line 65-68, `<p>Suggested actions</p>` rendered above every chip set
- This is a generic label that asserts AI involvement without naming the mechanism or showing any confidence signal. Per the Content Playbook, in-app copy should be informative, not pitchy. "Suggested actions" is the AI equivalent of "Smart features" -- it names the category without telling the user anything useful.
- The label is always present regardless of whether chips come from the AI API (`AISuggestionChips` with real confidence scores) or from static fallback suggestions. In the static case it is outright misleading since no AI suggestion has been made.
- Fix: For AI-sourced chips, either drop the label (the chips speak for themselves) or use something tied to the data: `"Based on similar messages"` or no label at all. For static fallback chips, either use a neutral label like `"Actions"` or suppress the label entirely.

**2. Chip tooltip text pattern `"Suggested: ${chip.reasoning}"`**

- Location: line 80, `title={chip.reasoning ? \`Suggested: ${chip.reasoning}\` : undefined}`
- The prefix `"Suggested: "` is redundant given the "Suggested actions" section header and the chip's role. The reasoning text itself (passed from the API) is not auditable from this file, but the prefix applies AI-assertion framing unconditionally. If the reasoning field contains weak or generic text from the API, the prefix amplifies it.
- Fix: Use `chip.reasoning` directly as the tooltip without the `"Suggested: "` prefix, or omit the tooltip prefix and let the reasoning stand on its own.

**3. Confidence percentage display may show "0%" for falsy values**

- Location: lines 85-89, `chip.confidence && chip.confidence > 0`
- This is a logic note: the condition `chip.confidence && chip.confidence > 0` correctly suppresses zero, but `chip.confidence` being `undefined` or `0` both return falsy and are handled the same way. This is correct behavior. No trope violation, noted for completeness.

**4. Static chip labels are functional and clean**

- `"Reply"`, `"Archive"`, `"Create TODO"`, `"Send reply"`, `"Edit reply"`, `"Archive source"`, `"View sent"`, `"Back to inbox"` -- all direct action verbs. No banned phrases. No overclaiming.

---

## Summary

| File                    | Violations                                                                                                                               |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| mobile-filter-sheet.tsx | None                                                                                                                                     |
| suggestion-chips.tsx    | 1 substantive (vague "Suggested actions" label, misleading in static fallback context), 1 minor (redundant "Suggested: " tooltip prefix) |

**Priority fixes:**

1. `suggestion-chips.tsx` line 65: replace or remove the "Suggested actions" label; differentiate behavior between AI-sourced and static-fallback chip sets
2. `suggestion-chips.tsx` line 80: drop the `"Suggested: "` prefix from the tooltip title; use `chip.reasoning` directly
