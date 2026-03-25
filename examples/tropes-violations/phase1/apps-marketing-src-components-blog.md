# Trope Audit: apps/marketing/src/components/blog/

Files audited:

- `apps/marketing/src/components/blog/blog-content.tsx`
- `apps/marketing/src/components/blog/blog-pagination.tsx`
- `apps/marketing/src/components/blog/blog-search.tsx`
- `apps/marketing/src/components/blog/copy-button.tsx`
- `apps/marketing/src/components/blog/social-share.tsx` — FILE DOES NOT EXIST

---

## blog-content.tsx

### Finding 1: Empty state copy — "Nothing here yet"

- **Location**: lines 119–121
- **Offending text**: `"Nothing here yet"` / `"Posts will show up here when they're published."`
- **Why it's noted**: This is a fine empty state — brief and honest. "Nothing here yet" is generic but also appropriate for an empty blog. Not a trope per se, but worth flagging in case brand voice should be applied here. No change required.

No other user-visible copy. Category labels ("Engineering", "Product", "Security", "Workflow"), search result counts, and "No posts found. Try a different search term." are all functional UI text with no trope risk.

**No significant findings.**

---

## blog-pagination.tsx

No user-visible copy beyond navigation labels ("Previous", "Next") and `aria-label` attributes. These are standard accessibility patterns with no trope risk.

**No findings.**

---

## blog-search.tsx

No user-visible copy beyond the placeholder `"Search posts..."` and the `aria-label` `"Clear search"`. Functional UI text only.

**No findings.**

---

## copy-button.tsx

No user-visible copy beyond `aria-label` values (`"Copied"`, `"Copy code"`). Functional UI text only.

**No findings.**

---

## social-share.tsx

File does not exist at the specified path. Cannot audit.

---

## Summary

No trope violations found in the blog components. The one empty-state string ("Nothing here yet") is borderline-generic but appropriate for the context. The missing `social-share.tsx` file could not be reviewed.
