# Tropes Audit: packages/ui/src/molecules/ContextBar

Files reviewed:

- `packages/ui/src/molecules/ContextBar/ContextBar.tsx`

---

## ContextBar.tsx

**"Context" label (line 68):**

> `<span className="font-medium text-muted-foreground text-[10px] uppercase tracking-wider">Context</span>`

The word "Context" appears as a section header above the chat context chip. This is product-specific terminology ("what the AI is aware of"), shown to users in the UI.

The word "Context" used as a UI label for an AI-awareness section is a mild AI-startup trope — it frames the AI's awareness in abstract terms rather than something concrete ("Viewing this email", "Focused on this task"). However, it is short, not hyperbolic, and may be intentional given the product positions around AI awareness.

Verdict: Minor. The label "Context" is vague where a more specific label would be clearer ("Viewing" or the message subject). Flag for potential improvement.

**JSDoc example (lines 12-18):**

```tsx
<ContextBar
  platform="google-gmail"
  sender="Sarah Chen"
  subject="Q4 Planning Discussion"
  preview="Hey team, I wanted to discuss our Q4 planning..."
  ...
/>
```

Example data. No trope language. No findings.

**Component description comment (lines 1-5):**

> "ContextBar - Displays current AI context above chat input"
> "Shows what content the AI is aware of (selected email, task, etc.) with an expandable preview section and dismiss functionality."

Internal developer documentation. The phrase "what content the AI is aware of" is abstract, but this is in a JSDoc comment, not user-visible copy. No action needed.

---

## Summary

One minor finding: the visible UI label `"Context"` (shown to users as a section header) is vague AI-framing language. A more specific label tied to the actual content type (e.g., "Viewing" + content title) would be clearer and less abstract.
