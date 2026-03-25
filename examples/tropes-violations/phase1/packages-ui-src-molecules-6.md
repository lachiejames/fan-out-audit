# Tropes Audit: packages/ui/src/molecules — Slice 6

Files reviewed:

- `packages/ui/src/molecules/input-group.tsx`
- `packages/ui/src/molecules/keybinding-hint.tsx`
- `packages/ui/src/molecules/label-group.tsx`
- `packages/ui/src/molecules/native-select.tsx`
- `packages/ui/src/molecules/pagination.tsx`
- `packages/ui/src/molecules/popover.tsx`
- `packages/ui/src/molecules/progress-bar.tsx`
- `packages/ui/src/molecules/select.tsx`
- `packages/ui/src/molecules/sheet.tsx`
- `packages/ui/src/molecules/tabs.tsx`
- `packages/ui/src/molecules/toggle-group.tsx`
- `packages/ui/src/molecules/chart.tsx`
- `packages/ui/src/molecules/chart.stories.tsx`

---

## input-group.tsx

No user-visible text. No findings.

---

## keybinding-hint.tsx

**JSDoc example (line 16):**

> `<KeybindingHint keys={['Cmd', 'K']} />` / `<KeybindingHint keys={['Ctrl', 'Shift', 'P']} />`

Key label strings. No trope language. No findings.

---

## label-group.tsx

No user-visible text (overflow label comes from `getOverflowLabel` utility). No findings.

---

## native-select.tsx

No user-visible text. No findings.

---

## pagination.tsx

**Aria labels (lines 50, 65, 98, 112, 128):**

> `"Go to first page"` / `"Go to previous page"` / `"Go to page ${page}"` / `"Go to next page"` / `"Go to last page"`

Accessible labels. Standard functional copy. No findings.

**Button text:**

> `"Previous"` / `"Next"`

No findings.

---

## popover.tsx

No user-visible text. No findings.

---

## progress-bar.tsx

**Aria label template (line 91):**

> `"AI confidence: ${clampedConfidence}%"` — wait, this is in ConfidenceBar, not ProgressBar.

ProgressBar itself: no user-visible text beyond `aria-label` passed in by the caller. No findings.

---

## select.tsx

No user-visible text. No findings.

---

## sheet.tsx

**Close button sr-only text (line 82):**

> `"Close"`

Accessibility text. No findings.

---

## tabs.tsx

No user-visible text. No findings.

---

## toggle-group.tsx

No user-visible text. No findings.

---

## chart.tsx

No user-visible text. Chart infrastructure only. No findings.

---

## chart.stories.tsx

**Data labels (lines 46-53):**

> `"Revenue"` / `"Expenses"` / `"Desktop"` / `"Mobile"` / `"Value"`

Generic data series labels for story demos. No AI trope language.

**Story descriptions (lines 63, 89, 115, 140, 182, 210, 235):**

> `"Basic line chart with two data series."` etc.

Technical developer descriptions. No findings.

---

## Summary

No AI writing tropes found in this slice.
