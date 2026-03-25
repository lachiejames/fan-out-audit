# Tropes Audit: packages/ui/src/atoms (Slice 164)

Files audited:

- `packages/ui/src/atoms/state-label.tsx`
- `packages/ui/src/atoms/switch.tsx`
- `packages/ui/src/atoms/text.tsx`
- `packages/ui/src/atoms/textarea.tsx`
- `packages/ui/src/atoms/tooltip.tsx`
- `packages/ui/src/atoms/toggle.tsx`

Note: `time-ago` and `toggle-group` were not found as separate files in this atoms directory. The `toggle.tsx` file covers the toggle atom.

---

## state-label.tsx

No findings. Status labels ("Draft", "Open", "Closed", "Merged", "In Progress", "Done", "Cancelled") are industry-standard workflow state names. No AI tropes.

---

## switch.tsx

No findings. No text content.

---

## text.tsx

No findings. Pure structural polymorphic text component. No hardcoded content.

---

## textarea.tsx

No findings. No hardcoded text content. Placeholders are passed by consumers.

---

## tooltip.tsx

No findings. No hardcoded text content. Tooltip text is always passed as children by consumers.

---

## toggle.tsx

No findings. No text content.

---

## Summary

**Total violations: 0**

All files in this slice are structural UI primitives. No user-facing copy present.
