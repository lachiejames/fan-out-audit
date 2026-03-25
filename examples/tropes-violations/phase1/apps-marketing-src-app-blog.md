# Tropes Audit: apps/marketing/src/app/blog (Slice 139)

**Files audited** (3):

- `blog/page.tsx`
- `blog/[slug]/layout.tsx` (file does not exist - 404 on read)
- `blog/[slug]/page.tsx`

---

## blog/page.tsx

User-visible copy:

**Page heading**: "Blog"

**Description** (rendered in `<p>` and repeated in metadata):

> "Notes on building SlopWeaver, working with AI tools, and managing context across too many apps."

**No tropes detected.** The description is first-person and specific. "Managing context across too many apps" is a genuine pain point stated plainly, not inflated. No magic adverbs, no ornate nouns, no rhetorical questions, no anaphora, no AI cliches.

No findings.

---

## blog/[slug]/layout.tsx

File does not exist at the audited path. No findings.

---

## blog/[slug]/page.tsx

This file is a dynamic rendering shell. All prose copy is sourced from MDX blog post frontmatter (`post.title`, `post.description`, `post.author`, `post.tags`) and injected at render time - none of it is authored here.

**Static UI labels rendered in this file:**

- "Back to Blog"
- "Related Posts"
- "All posts"

These are navigation labels, not prose copy. No tropes applicable.

The `CATEGORY_LABELS` map contains: "Engineering", "Product", "Security", "Workflow" - plain category names, no tropes.

No findings.

---

## Summary

No trope violations found across the 2 existing files. The only authored copy in this slice is the blog page description, which is plain and specific. Blog post content itself is in MDX files (not audited in this slice).
