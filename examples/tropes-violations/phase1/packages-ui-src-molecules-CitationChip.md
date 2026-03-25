# Tropes Audit: packages/ui/src/molecules/CitationChip

Files reviewed:

- `packages/ui/src/molecules/CitationChip/CitationChip.tsx`

---

## CitationChip.tsx

**"View original" action label (line 147):**

> `"View original"`

Plain functional label. No findings.

**Fallback author label (line 41):**

> `return author ?? citation.title ?? "Unknown";`

`"Unknown"` is a reasonable fallback for missing metadata. Not a trope.

**JSDoc example (lines 65-70):**

```tsx
<CitationChip
  citation={{ id: "1", text: "Project deadline", source: "...", platform: "slack", sender: "Sarah", date: "Jan 15" }}
  number={1}
  onClick={() => openCitationPanel("1")}
/>
```

Example data: `"Project deadline"` as citation text. Neutral, functional. No findings.

**Component description comment (lines 54-61):**

> "Shows citation number and platform icon inline in AI responses."
> "Hover reveals a detailed preview card with: Platform-specific header with icon and label, Sender name and date, Preview/source text (truncated to 3 lines), 'View original' action button"

Internal developer documentation. Not user-facing copy. No findings.

---

## Summary

No AI writing tropes found. CitationChip has minimal user-visible text ("View original" action label and "Unknown" fallback) and no trope language.
