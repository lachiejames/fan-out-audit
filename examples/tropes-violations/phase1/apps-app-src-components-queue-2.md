# Trope Audit — apps/app/src/components/queue (Slice 93)

Files audited:

- queue-filter-dropdown.tsx
- queue-skeleton.tsx
- queue-slide-over-calendar-edit-fields.tsx
- queue-slide-over-edit-section.tsx
- queue-slide-over-footer.tsx
- queue-slide-over-header.tsx
- queue-slide-over-terminal-info.tsx
- queue-slide-over.tsx

---

## Findings

### queue-slide-over-footer.tsx

**Lines 134-138 — AI disclaimer paragraph**

```tsx
<p className="mt-2 text-muted-foreground text-xs leading-relaxed">
  AI-generated drafts are suggestions only. Every draft requires explicit human approval before sending. The AI has no
  ability to send emails, access credentials, or take actions autonomously. We validate all AI outputs for injection
  artifacts and PII. However, no AI system is immune to adversarial manipulation — always review drafts before
  approving.
</p>
```

This is a dense legal/compliance disclaimer embedded in the primary action footer of every single queue item. It has several problems:

1. "AI-generated drafts are suggestions only" — restates the UI structure the user is already looking at. The approve button exists precisely because drafts require approval.
2. "The AI has no ability to send emails, access credentials, or take actions autonomously" — directly contradicts the product's core value proposition. SlopWeaver's selling point is that it _does_ take actions (with approval). The disclaimer is written as if the product does none of that.
3. "We validate all AI outputs for injection artifacts and PII" — internal engineering concern stated as user-facing reassurance. Users do not know what "injection artifacts" means and should not need to.
4. "no AI system is immune to adversarial manipulation" — boilerplate safety hedging. Appropriate in a terms-of-service doc, not in a footer printed on every action card.
5. The full block reads as AI-product liability copy pasted into the UI, not as something a real product team decided users need to read on every approval. It undermines confidence rather than building it.

**Recommendation**: Remove or drastically shorten. If a disclaimer is required for legal reasons, move it to settings or a first-use modal. The footer already has an approve button with an explicit label — the affordance is self-explanatory.

---

### queue-slide-over.tsx

**Line 260 — "AI will learn from this" toast**

```tsx
toast.success(feedback ? "Thanks, AI will learn from this" : "Action discarded");
```

"AI will learn from this" is a common reassurance trope in AI products. It promises a feedback loop without committing to any observable outcome. If the system genuinely retrains or adjusts behavior from this signal, say what changes. If it just records the feedback, "Feedback saved" is more honest. The phrase "AI will learn" anthropomorphizes the system and has become meaningless through repetition across the industry.

---

### queue-slide-over-terminal-info.tsx

No trope violations. The terminal state descriptions ("This action was discarded and will not be executed", "This action expired before it could be executed") are plain and accurate.

---

### queue-filter-dropdown.tsx, queue-skeleton.tsx, queue-slide-over-calendar-edit-fields.tsx, queue-slide-over-edit-section.tsx, queue-slide-over-header.tsx

No findings. These files contain UI structure, form fields, and interaction logic with no user-visible copy that exhibits AI writing tropes.

---

## Summary

| File                        | Violation                                                      | Severity |
| --------------------------- | -------------------------------------------------------------- | -------- |
| queue-slide-over-footer.tsx | Multi-sentence AI disclaimer block in every action footer      | High     |
| queue-slide-over.tsx        | "AI will learn from this" anthropomorphizing reassurance toast | Medium   |
