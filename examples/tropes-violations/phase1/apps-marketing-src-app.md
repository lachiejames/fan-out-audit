# Tropes Audit: apps/marketing/src/app (Slice 138)

**Files audited** (8):

- `global-error.tsx`
- `layout.tsx`
- `not-found.tsx`
- `opengraph-image.tsx`
- `page.tsx`
- `posthog-provider.tsx`
- `robots.ts`
- `sitemap.ts`

---

## global-error.tsx

No user-visible copy. Pure error-capture wrapper. No findings.

---

## layout.tsx

User-visible copy is limited to metadata strings repeated across `description`, `openGraph.description`, and `twitter.description`:

> "SlopWeaver connects 17 work tools, learns your patterns, and proves it with metrics. Track acceptance rates, confidence scores, and time saved."

**No tropes detected.** The copy is concrete, metric-first, and avoids ornate nouns, magic adverbs, and AI-writing cliches. "Proves it with metrics" is specific and on-brand.

`openGraph.title` / `twitter.title`: "SlopWeaver - AI that proves it's learning" - clean, no tropes.

No findings.

---

## not-found.tsx

Copy:

> "This page doesn't exist. It may have been moved or removed."

**No tropes detected.** Plain, functional, accurate.

CTA: "Back to home" - fine.

No findings.

---

## opengraph-image.tsx

Rendered text in the OG image:

- "SlopWeaver" (brand name)
- "AI that proves it's learning" (tagline)
- "Acceptance rates · Confidence scores · Time saved" (value props)

**No tropes detected.** The value props line is a concrete noun list with no filler. No ornate language, no AI writing cliches.

No findings.

---

## page.tsx

This file is a pure component composition (imports and renders section components). Contains no authored prose copy. No findings.

---

## posthog-provider.tsx

Contains only code comments and JSDoc. No user-visible copy. No findings.

---

## robots.ts

Contains only code comments and configuration values (URLs, Next.js doc references). No user-visible copy. No findings.

---

## sitemap.ts

Contains only code comments and configuration data (URLs, priorities, change frequencies). No user-visible copy. No findings.

---

## Summary

No trope violations found across all 8 files. The only user-visible copy in this slice (metadata descriptions, OG text, 404 message) is concise, concrete, and clean.
