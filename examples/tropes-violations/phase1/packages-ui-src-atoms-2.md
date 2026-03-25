# Tropes Audit: packages/ui/src/atoms (Slice 162)

Files audited:

- `packages/ui/src/atoms/counter-label.tsx`
- `packages/ui/src/atoms/details.tsx`
- `packages/ui/src/atoms/heading.tsx`
- `packages/ui/src/atoms/hidden.tsx`
- `packages/ui/src/atoms/icon.tsx` (icon wrapper -- no `indicator` file found; `icon-button` not present in atoms dir)
- `packages/ui/src/atoms/input.tsx`
- `packages/ui/src/atoms/label.tsx`

Note: `indicator` and `icon-button` were not found as separate files in this directory. The `icon.tsx` file covers the icon wrapper atom.

---

## counter-label.tsx

No findings. No user-visible text beyond formatting logic. JSDoc example shows generic numeric usage.

---

## details.tsx

No findings. The only hardcoded text is the default `summary` prop value `"Details"` -- a single functional English word used as a fallback when no custom summary is provided. Not AI copy.

---

## heading.tsx

No findings. Pure structural component. No text content.

---

## hidden.tsx

No findings. No text content. Comment strings like `"Desktop/tablet only content"` and `"Mobile/tablet only content"` in JSDoc examples are developer documentation, not user-facing copy.

---

## icon.tsx

No findings. No text content.

---

## input.tsx

No findings. The `loadingText` default value is `"Loading"` -- a single functional word for screen readers. No placeholder text is hardcoded; placeholders are passed by consumers. The `"use client"` directive is a Next.js pattern artifact.

---

## label.tsx

No findings. No text content.

---

## Summary

**Total violations: 0**

All files in this slice are structural UI primitives with no user-facing copy containing AI writing tropes.
