# Tropes Audit — apps/app/src/components/citations

Files audited:

- citation-chip.tsx

## Findings

### citation-chip.tsx — line 107

**File**: `apps/app/src/components/citations/citation-chip.tsx`
**Line**: 107
**Trope**: Boilerplate "From:" prefix label in secondary metadata line.

```tsx
const secondaryLine = author && author !== primaryLine ? `From: ${author}` : "";
```

**Problem**: "From: [author]" is a minimal, AI-default way to surface authorship. The "From:" prefix is the most generic possible framing. Depending on the citation type (email, doc, Slack message) "from" may be the wrong word entirely (e.g. a Google Doc has an "author" or "owner", not a sender). This is a low-severity trope — the string is functional — but the blanket "From:" label applied to all platforms regardless of context is the kind of lazy default AI copy produces.

**Suggested fix**: Either use a platform-aware label (e.g. "By" for documents, "From" for messages) or drop the prefix and just render the author name directly, since the context of a citation card already implies attribution.

---

No other findings in this file.
