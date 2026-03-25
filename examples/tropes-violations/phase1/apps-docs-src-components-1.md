# Trope Audit: apps/docs/src/components/docs (Slice 133)

**Files audited:**

- `apps/docs/src/components/docs/callout.tsx`
- `apps/docs/src/components/docs/code-block.tsx`
- `apps/docs/src/components/docs/docs-breadcrumb-jsonld.tsx`
- `apps/docs/src/components/docs/docs-search.tsx`
- `apps/docs/src/components/docs/integration-page-template.tsx`

**Note:** The originally listed files `features-list.tsx`, `footer.tsx`, `header.tsx`, and `hero-section.tsx` do not exist in the repository. The actual component directory is `apps/docs/src/components/docs/` (not `apps/docs/src/components/`). Audited the five files that do exist within this slice's directory.

---

## callout.tsx

No user-visible prose. Pure structural/layout component with icon and style configuration. No findings.

---

## code-block.tsx

No user-visible prose. UI component only. No findings.

---

## docs-breadcrumb-jsonld.tsx

Contains navigation title strings (`"Introduction"`, `"AI Chat"`, `"Quick Start"`, etc.) and the label `"Docs"`. These are proper nouns and navigation labels, not marketing copy. No findings.

---

## docs-search.tsx

Contains user-visible text strings. Findings below.

**VIOLATION — Filler excerpt copy:**

File: `apps/docs/src/components/docs/docs-search.tsx`, line 21 (mockResults array)

```
excerpt: "Get up and running with SlopWeaver quickly...",
```

"Get up and running" is a stock phrase. Minor, but `mockResults` is visible UI copy shown in the search modal.

```
excerpt: "Master SlopWeaver with keyboard shortcuts...",
```

"Master" is a common AI-copy inflation verb implying transformative power from a mundane feature.

```
excerpt: "Chat with your workspace assistant...",
```

"Workspace assistant" is generic AI-product phrasing. Vague.

**Severity:** Low. These excerpts appear only in the search fallback/mock results, not primary product copy. Still worth fixing to maintain copy consistency.

**Suggested replacements:**

- "Get up and running with SlopWeaver quickly..." → "Connect your tools and configure sync in under five minutes."
- "Master SlopWeaver with keyboard shortcuts..." → "Keyboard shortcuts for inbox, queue, and search."
- "Chat with your workspace assistant..." → "Ask questions, draft replies, and run actions from the AI panel."

---

## integration-page-template.tsx

Contains one piece of user-visible prose in a shared callout rendered on every integration page (line 201-206):

```
Data from this integration feeds the AI's learning. Messages, tasks, and activity synced here
contribute to your behavioral fingerprint and improve draft accuracy over time.
```

**Assessment:** This is factual product description, not trope-heavy. "Feeds the AI's learning" is plain language. "Behavioral fingerprint" is a named product concept, not a buzzword used here. "Improve draft accuracy over time" is a concrete claim. No violation.

**No findings** in this file beyond the docs-search issues above.
