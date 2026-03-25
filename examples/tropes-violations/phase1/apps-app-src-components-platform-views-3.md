# Tropes Audit: apps/app/src/components/platform-views (Slice 91)

Files audited:

- linkedin-post-view.tsx
- monday-item-view.tsx
- notion-page-view.tsx
- platform-view-shared.tsx
- resource-card-view.tsx
- slack-reaction-picker.tsx
- slack-thread-view.tsx
- teams-thread-view.tsx

---

## Findings

### 1. `resource-card-view.tsx` — subtitle "Resource preview"

**File**: `apps/app/src/components/platform-views/resource-card-view.tsx`
**Line**: 84
**Text**: `"Resource preview"`

**Why it's a trope**: Placeholder-grade descriptor. "Resource preview" is what a developer writes when they need a subtitle but have no copy direction. It communicates nothing to the user about what they're looking at or why it matters.

**Fix direction**: Remove entirely, or replace with something descriptive of the content type (e.g. "Document summary" for Google Docs, "File" for Drive/OneDrive). The `config.label` already identifies the platform; this subtitle adds no value.

---

### 2. `monday-item-view.tsx` — empty activity state "No updates loaded."

**File**: `apps/app/src/components/platform-views/monday-item-view.tsx`
**Line**: 83
**Text**: `"No updates loaded."`

**Why it's a trope**: Passive, developer-facing empty state. "Not loaded" implies a system failure or an unfinished feature rather than a genuine empty state. Users read "loaded" as an implementation detail leaking through.

**Fix direction**: "No updates yet" or simply omit the section when there are no updates.

---

### 3. `monday-item-view.tsx` — composer placeholder "Write an update... (Cmd+Enter to post)"

**File**: `apps/app/src/components/platform-views/monday-item-view.tsx`
**Line**: 107
**Text**: `"Write an update... (Cmd+Enter to post)"`

**Why it's a trope**: Stock composer placeholder. Functional, but generic across every task management tool. The keyboard shortcut hint is useful and should be kept regardless of whether the preamble changes.

**Fix direction**: Low priority. Acceptable as-is given the shortcut hint.

---

### 4. `notion-page-view.tsx` — section heading "Page content"

**File**: `apps/app/src/components/platform-views/notion-page-view.tsx`
**Line**: 125
**Text**: `"Page content"`

**Why it's a trope**: Tautological section label. When viewing a Notion page, a section called "Page content" names what every user already knows they're looking at. The label wastes the heading space.

**Fix direction**: Remove the heading and let the content stand alone, or replace with something useful like the page title (already shown in the header) or a doc-type hint.

---

### 5. `notion-page-view.tsx` — empty state "No page content available."

**File**: `apps/app/src/components/platform-views/notion-page-view.tsx`
**Line**: 129
**Text**: `"No page content available."`

**Why it's a trope**: Passive filler. Consistent with the "No X available" pattern found throughout these views.

**Fix direction**: "This page appears to be empty" or "Nothing to show — open in Notion to see the full page."

---

### 6. `linkedin-post-view.tsx` — empty state "Post preview not available."

**File**: `apps/app/src/components/platform-views/linkedin-post-view.tsx`
**Line**: 55
**Text**: `"Post preview not available."`

**Why it's a trope**: Passive "not available" empty state. Same pattern as above.

**Fix direction**: "Open on LinkedIn to read the full post" (the open button is already rendered nearby, so this can point at it).

---

### 7. `slack-thread-view.tsx` — toast "Missing thread context for reply"

**File**: `apps/app/src/components/platform-views/slack-thread-view.tsx`
**Line**: 129
**Text**: `"Missing thread context for reply"`

**Why it's a trope**: Developer-facing error language. "Thread context" is an implementation concept, not user vocabulary. Users do not know what "context" means here and cannot act on the message.

**Fix direction**: "Can't reply — the original thread wasn't found. Open in Slack to reply directly."

---

### 8. `slack-thread-view.tsx` — generic "Failed to send reply" / "Failed to add reaction"

**File**: `apps/app/src/components/platform-views/slack-thread-view.tsx`
**Lines**: 153, 370
**Text**: `"Failed to send reply"`, `"Failed to add reaction"`

**Why it's a trope**: Generic failure toasts with no actionability. Both appear frequently across the codebase and provide no path forward.

**Fix direction**: Add minimal context. "Couldn't send — check your Slack connection" or simply retry the action inline. Not urgent individually but worth addressing as part of a broader error-copy pass.

---

### 9. `teams-thread-view.tsx` — generic "Failed to send reply"

**File**: `apps/app/src/components/platform-views/teams-thread-view.tsx`
**Line**: 153
**Text**: `"Failed to send reply"`

Same pattern as the Slack equivalent above.

---

## Summary

| #   | File                   | Line(s)  | Offending text                                    | Category                            |
| --- | ---------------------- | -------- | ------------------------------------------------- | ----------------------------------- |
| 1   | resource-card-view.tsx | 84       | "Resource preview"                                | Placeholder-grade subtitle          |
| 2   | monday-item-view.tsx   | 83       | "No updates loaded."                              | Implementation-detail empty state   |
| 3   | monday-item-view.tsx   | 107      | "Write an update..."                              | Stock composer placeholder          |
| 4   | notion-page-view.tsx   | 125      | "Page content"                                    | Tautological section label          |
| 5   | notion-page-view.tsx   | 129      | "No page content available."                      | Passive "not available" empty state |
| 6   | linkedin-post-view.tsx | 55       | "Post preview not available."                     | Passive "not available" empty state |
| 7   | slack-thread-view.tsx  | 129      | "Missing thread context for reply"                | Developer-facing error copy         |
| 8   | slack-thread-view.tsx  | 153, 370 | "Failed to send reply" / "Failed to add reaction" | Generic failure toast               |
| 9   | teams-thread-view.tsx  | 153      | "Failed to send reply"                            | Generic failure toast               |

No AI writing tropes (over-claiming, "seamlessly", "powerful", "revolutionize", etc.) were found in this slice. Issues are limited to stock UX copy, passive empty states, and developer-facing error messages.
