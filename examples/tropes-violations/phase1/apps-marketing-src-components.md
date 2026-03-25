# Trope Audit: apps/marketing/src/components/

Files audited:

- `apps/marketing/src/components/feature-section.tsx`
- `apps/marketing/src/components/legal-section.tsx`
- `apps/marketing/src/components/page-header.tsx`

---

## feature-section.tsx

No user-visible copy. This is a pure layout/rendering component. It accepts `title`, `description`, and `features` as props and renders them; it contains no hardcoded copy. Trope risk is zero at the component level — any tropes will be in the consuming pages that pass the props.

**No findings.**

---

## legal-section.tsx

No user-visible copy. This is a pure layout/rendering component for legal pages. It renders children and a title prop. No hardcoded copy.

**No findings.**

---

## page-header.tsx

No user-visible copy. This is a layout component that renders a `title` and optional `subtitle` prop inside a styled section. No hardcoded copy.

**No findings.**

---

## Summary

No trope violations found in these three shared components. All copy responsibility is delegated to props passed by consuming pages.
