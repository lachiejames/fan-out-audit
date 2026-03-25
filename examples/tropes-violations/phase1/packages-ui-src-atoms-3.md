# Tropes Audit: packages/ui/src/atoms (Slice 163)

Files audited:

- `packages/ui/src/atoms/progress.tsx`
- `packages/ui/src/atoms/scroll-area.tsx`
- `packages/ui/src/atoms/separator.tsx`
- `packages/ui/src/atoms/skeleton.stories.tsx`
- `packages/ui/src/atoms/skeleton.tsx`
- `packages/ui/src/atoms/spinner.tsx`
- `packages/ui/src/atoms/stack-item.tsx` (found at `foundations/layout/stack-item.tsx`)

Note: `sr-only` was not found as a separate file in this atoms directory. `stack-item` lives at `foundations/layout/stack-item.tsx`.

---

## progress.tsx

No findings. No text content.

---

## scroll-area.tsx

No findings. No text content.

---

## separator.tsx

No findings. No text content.

---

## skeleton.stories.tsx

No findings. Story names and descriptions are technical/factual: "Default skeleton with pulse animation", "Static skeleton without animation", "Circular skeleton, commonly used as avatar placeholder", "Skeleton card - typical loading state for a card component", "Profile skeleton - loading state for a user profile", "Text block skeleton - loading state for paragraph text", "Table row skeleton - loading state for table data". All are accurate functional descriptions.

---

## skeleton.tsx

No findings. The JSDoc example comment `"Loading:"` is a code snippet, not user-facing copy. No hardcoded user-visible text.

---

## spinner.tsx

No findings. The `srText` default is `"Loading"` -- a single functional word for screen readers.

---

## stack-item.tsx (foundations/layout)

No findings. No text content. JSDoc usage example text ("Fixed width", "Flexible width") is developer documentation.

---

## Summary

**Total violations: 0**

These are loading state and layout primitives. No user-facing copy present.
