# Trope Audit — apps/app/src/components/queue (Slice 92)

Files audited:

- queue-ai-transparency-summary.tsx
- queue-audit-detail-renderer.tsx
- queue-audit-log-view.tsx
- queue-audit-log.tsx
- queue-card.tsx
- queue-draft-block.tsx
- queue-empty-state.tsx
- queue-filter-bar.tsx

---

## Findings

### queue-ai-transparency-summary.tsx

**Line 51 — "AI Trust" label**

```tsx
<span className="font-medium">AI Trust</span>
```

"AI Trust" as a section heading is an abstract concept dressed up as a feature. Users don't interact with "trust" as a noun. The label names an internal framing ("we built a trust layer") rather than something the user can act on or verify. Consider a concrete heading like "How this was generated" or simply removing the heading and letting the data speak.

---

### queue-draft-block.tsx

**Line 109 — "Based on" citation preamble**

```tsx
<span className="text-muted-foreground text-xs">Based on</span>
```

"Based on" is vague. It implies sources were consulted without saying what role they played. This is fine if it's a literal description (the draft is based on these emails/documents), but if the citations are only loosely related context, the label overstates the connection.

**Line 152 — "Why this draft?" toggle**

```tsx
Why this draft?
```

Acceptable phrasing. Direct question the user would actually ask. No issue here.

---

### queue-empty-state.tsx

**Line 68 — "All caught up!"**

```tsx
<h2 className="mb-2 font-bold text-2xl text-foreground">All caught up!</h2>
```

"All caught up!" is a stock empty-state trope lifted from every email and todo app built since 2015. It is cheerful noise that adds nothing. If the queue is empty because the AI found nothing to propose, that's worth saying plainly. If the user has processed everything, say that. The exclamation mark compounds the problem.

**Line 69-71 — Empty state body copy**

```tsx
No pending approvals. As SlopWeaver learns your preferences, AI-suggested actions will appear here with
confidence scores.
```

"As SlopWeaver learns your preferences" is product-marketing copy inside the product itself. It doesn't help a user who is staring at an empty queue right now. It reads like an onboarding tooltip that was never removed. The second sentence ("AI-suggested actions will appear here with confidence scores") is redundant — the user is already on the queue page and has presumably seen actions before.

**Line 98 — "Ask a question" dead link**

```tsx
<Link to="#">
  ...
  Ask a question
```

`to="#"` is a broken link rendered as a primary affordance. This is not a trope per se, but it is misleading UI that suggests a capability that does not exist. The button does nothing.

---

### queue-filter-bar.tsx

**Line 147 — Subtitle copy**

```tsx
<p className="mt-1 text-muted-foreground text-sm">Proposed actions ready for your review</p>
```

"Proposed actions ready for your review" restates what the page heading "Queue" already communicates. It is filler that uses the AI-marketing passive construction ("proposed by" is implied but omitted, softening agency). A tighter version: "Actions awaiting approval" or nothing.

---

## Summary

| File                              | Violation                                                         | Severity |
| --------------------------------- | ----------------------------------------------------------------- | -------- |
| queue-ai-transparency-summary.tsx | "AI Trust" abstract section label                                 | Low      |
| queue-draft-block.tsx             | "Based on" vague citation preamble                                | Low      |
| queue-empty-state.tsx             | "All caught up!" stock empty-state cliche                         | Medium   |
| queue-empty-state.tsx             | "As SlopWeaver learns your preferences" in-product marketing copy | Medium   |
| queue-empty-state.tsx             | "Ask a question" dead link (`to="#"`)                             | Medium   |
| queue-filter-bar.tsx              | "Proposed actions ready for your review" redundant subtitle       | Low      |
