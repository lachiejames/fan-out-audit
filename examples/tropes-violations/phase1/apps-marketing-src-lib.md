# Trope Audit: apps/marketing/src/lib/blog.ts (Slice 159)

File audited:

- blog.ts

---

## Findings

No findings.

The file is a utility library for reading MDX blog posts from the filesystem. It contains no user-facing copy. All strings are technical identifiers:

- File extension filters (`".mdx"`)
- Default fallback values for missing frontmatter (`"Lachie James"`, `"engineering"`, `"productivity"`)
- No marketing copy, no headings, no descriptive text

The JSDoc comments (`"Get all published blog posts sorted by date (newest first)"`, `"Get a single blog post by slug"`, etc.) are internal developer documentation and are outside the scope of this audit.
