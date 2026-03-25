# Tropes Audit: packages/ui/src/molecules/ConfidenceBar

Files reviewed:

- `packages/ui/src/molecules/ConfidenceBar/ConfidenceBar.tsx`
- `packages/ui/src/molecules/ConfidenceBar/ConfidenceBar.stories.tsx`

---

## ConfidenceBar.tsx

**Aria label (line 90):**

> `aria-label={\`AI confidence: ${clampedConfidence}%\`}`

The phrase "AI confidence" is visible to screen readers and surfaces the AI framing directly in the accessibility tree. This is accurate product language (SlopWeaver explicitly shows confidence scores as a differentiator — "The AI that proves it's learning you"), so it is intentional and on-brand. However, the word "AI" as a prefix is worth noting.

Verdict: Borderline. This is intentional product language, not a vague AI-startup trope. The metric is real and exposed to users. No change recommended, but flag for awareness.

**SuggestionType values (type definition, line 6):**

> `"urgent_mention"` / `"hidden_todo"` / `"awaiting_response"` / `"deadline"` / `"cross_platform"`

Internal enum values, not user-visible strings. No findings.

---

## ConfidenceBar.stories.tsx

**Story labels in render functions (lines 214-228):**

> `"Low (25%)"` / `"Medium (55%)"` / `"High (85%)"` / `"Very High (98%)"`

Developer story labels, not production copy. No findings.

**Story labels for suggestion types (lines 233-259):**

> `"Urgent Mention"` / `"Hidden TODO"` / `"Awaiting Response"` / `"Deadline"` / `"Cross Platform"`

Developer story labels. No findings.

**Argtype descriptions (lines 6-36):**

> `"Whether to animate the bar fill on mount"` / `"Color mode: type-based or confidence-based"` / `"Confidence percentage (0-100)"` / `"Custom label (overrides percentage display)"` etc.

Storybook controls documentation for developers. No findings.

---

## Summary

One borderline finding: the aria-label `"AI confidence: X%"` surfaces "AI" directly to screen readers. This is intentional and accurate product language rather than a trope, but it is worth awareness during copy review. No other findings.
