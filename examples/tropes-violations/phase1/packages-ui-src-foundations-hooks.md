# Tropes Audit: packages/ui/src/foundations/hooks (Slice 173)

Files audited:

- `packages/ui/src/foundations/hooks/use-mobile.tsx`
- `packages/ui/src/foundations/hooks/use-password-visibility.tsx`
- `packages/ui/src/foundations/hooks/use-escape-key.tsx`
- `packages/ui/src/foundations/hooks/use-media.tsx`
- `packages/ui/src/foundations/hooks/match-media.tsx`
- `packages/ui/src/foundations/hooks/use-responsive-value.tsx`

---

## use-mobile.tsx

No findings. Pure logic hook. JSDoc example uses `<MobileView />` and `<DesktopView />` as developer placeholders.

---

## use-password-visibility.tsx

No findings. Pure logic hook. JSDoc example button text `"Hide"` / `"Show"` is developer placeholder code, not product copy.

---

## use-escape-key.tsx

No findings. Pure logic hook. No text content.

---

## use-media.tsx

No findings. Pure logic hook. JSDoc examples use media query strings as technical identifiers. The `shallowEqual` JSDoc example uses generic key-value pairs.

---

## match-media.tsx

No findings. Context provider component. JSDoc example uses `"(pointer: coarse)"` as a technical media query string.

---

## use-responsive-value.tsx

No findings. Pure logic hook. All content is TypeScript type definitions and technical comments.

---

## Summary

**Total violations: 0**

All hook files are pure logic with no user-facing text. This was expected.
