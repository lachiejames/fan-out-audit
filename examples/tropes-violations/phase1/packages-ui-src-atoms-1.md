# Tropes Audit: packages/ui/src/atoms (Slice 161)

Files audited:

- `packages/ui/src/atoms/PriorityBadge.tsx`
- `packages/ui/src/atoms/avatar-stack.tsx`
- `packages/ui/src/atoms/avatar.tsx`
- `packages/ui/src/atoms/badge.tsx`
- `packages/ui/src/atoms/button.tsx`
- `packages/ui/src/atoms/checkbox.tsx`
- `packages/ui/src/atoms/context-chip.tsx`

---

## PriorityBadge.tsx

No findings. Pure display component. Labels ("Critical", "High", "Medium", "Low") are factual priority tier names. JSDoc example `reasoning="VIP contact, urgent language"` is a developer-facing code comment, not user-visible copy.

---

## avatar-stack.tsx

No findings. No user-visible text. The "+N" overflow indicator is a universally understood UI convention.

---

## avatar.tsx

No findings. Pure structural component. No text content.

---

## badge.tsx

No findings. Pure variant configuration. No hardcoded text content.

---

## button.tsx

No findings. The `sr-only` loading text is `"Loading"` -- a single functional word for screen readers, not marketing copy.

---

## checkbox.tsx

No findings. No user-visible text.

---

## context-chip.tsx

No findings. Aria labels are functional ("Collapse preview", "Expand preview", "Clear context"). The JSDoc example uses `sender="Sarah Chen"` and `subject="Q4 Planning Discussion"` as placeholder data for developers -- not user-facing copy.

---

## Summary

**Total violations: 0**

These are all structural UI primitives. No labels, placeholders, aria-labels, or story descriptions contain AI writing tropes.
