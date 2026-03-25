# Tropes Audit: packages/ui/src/molecules/SuggestionCard

Files reviewed:

- `packages/ui/src/molecules/SuggestionCard/SuggestionCard.tsx`

---

## SuggestionCard.tsx

**JSDoc example (lines 83-95):**

```tsx
suggestion={{
  id: "1",
  type: "urgent_mention",
  title: "Urgent Mention Detected",
  description: "CEO mentioned 'critical' in latest email",
  confidence: 95,
  action: "view_email",
  actionLabel: "View Email",
}}
```

The title `"Urgent Mention Detected"` uses AI-detection framing ("Detected") that is passive and algorithmic. The description `"CEO mentioned 'critical' in latest email"` is actually concrete and specific — this is the kind of copy that works.

Verdict: The JSDoc example `"Urgent Mention Detected"` as a title would be a violation if it appears in production. The word "Detected" frames the AI's work as surveillance/scanning. Better: name what the user should do, not what the AI found — e.g., `"Reply needed from CEO"`. However, this is only in JSDoc example data, not a hardcoded string.

**Snooze button title (line 181):**

> `title="Snooze suggestion"`

Functional tooltip. No findings.

**Dismiss button title (line 129):**

> `title="Dismiss"`

Functional tooltip. No findings.

**Related items label (line 143):**

> `"Related across platforms:"`

Plain functional label. No findings.

**SuggestionType enum values (line 6, from ConfidenceBar import):**

> `"urgent_mention"` / `"hidden_todo"` / `"awaiting_response"` / `"deadline"` / `"cross_platform"`

Internal type values, not user-visible strings directly. However, these types will map to titles and action labels that are passed in by the caller. The component itself only renders `suggestion.title` and `suggestion.description` which are caller-provided. No findings in the component itself.

**Component description comment (lines 72-81):**

> "SuggestionCard - Suggestion card with type-based styling"
> The JSDoc `@example` block references `"Urgent Mention Detected"` as a suggested title.

As noted above, the example title contains AI-detection framing. This seeds the pattern for callers.

---

## Summary

One borderline finding: the JSDoc example title `"Urgent Mention Detected"` models AI-detection framing language for component consumers. While not a hardcoded user-visible string in the component itself, it establishes a copy pattern that callers may follow. The `"Detected"` suffix is passive algorithmic framing — callers should be guided toward action-oriented titles instead (e.g., "Reply needed", "Deadline approaching", "Message awaiting response").

No hardcoded user-visible trope violations in the component implementation itself.
