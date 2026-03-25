# Tropes Audit: apps/app/src/components (batch 1)

Files audited:

- `action-meter-mini.tsx`
- `ai-chat-action-button.tsx`
- `ai-chat-message-content.tsx`
- `ai-chat-message.tsx`
- `ai-chat-reasoning-block.tsx`
- `ai-chat-streaming-cursor.tsx`
- `ai-chat-timeline.tsx`
- `ai-chat-tool-approval-actions.tsx`

---

## Findings

### `ai-chat-reasoning-block.tsx`, line 38

**Trope: serves-as dodge / ornate framing for a simple conditional**

Text: `"Available only when the model uses extended reasoning."`

This sentence is shown when the reasoning block is collapsed and reasoning is not yet streaming. It reads like a tooltip that explains a product concept rather than telling the user what to do. It also hedges with "available only when" instead of stating the current state directly. Acceptable rewrite: `"Shown when extended reasoning is on."` or simply nothing (the section header "Reasoning" with a "Show" button is self-explanatory).

Severity: minor.

---

### `ai-chat-timeline.tsx`, line 125

**Trope: rhetorical / open-ended question as empty-state heading**

Text: `"What do you want to do?"`

Empty-state headings phrased as rhetorical questions are a well-worn AI product cliche (ChatGPT, Claude, Gemini all use variants). The question implies the product is infinitely capable; it also puts cognitive load on the user rather than orienting them. A concrete alternative: `"How can I help?"` is marginally better but equally overused. The cleanest fix is a short directive statement that reflects SlopWeaver's actual pitch, e.g. `"Ask me anything about your work."` or a product-specific prompt.

Severity: moderate.

---

### `ai-chat-timeline.tsx`, line 126

**Trope: vague capability claim / filler description**

Text: `"I can help with messages, tasks, replies, and more."`

"...and more" is a stock filler phrase that adds no information. The list itself (messages, tasks, replies) is generic enough to describe every AI assistant. If this text is meant to orient the user, it should reflect what SlopWeaver specifically does that others don't. Alternatively, drop the description and let the suggestion chips below do the work.

Severity: minor.

---

### `ai-chat-tool-approval-actions.tsx`, line 112

**Trope: rhetorical question as micro-copy**

Text: `"Approve this action?"`

Phrasing an approval prompt as a question (instead of a statement or imperative) is soft and ambiguous. The button label "Approve" already communicates the action. A direct label like `"Action needs approval"` or just the tool name with "Approve / Deny" buttons is cleaner and less AI-cliche.

Severity: minor.

---

## No findings

- `action-meter-mini.tsx` - UI-only, no user-facing prose.
- `ai-chat-action-button.tsx` - No user-facing text beyond button titles ("Copy", "Regenerate", "Good response", "Poor response") which are functional labels, not prose.
- `ai-chat-message-content.tsx` - No static user-facing text.
- `ai-chat-streaming-cursor.tsx` - No user-facing text.
- `ai-chat-message.tsx` - The only static text is `"Action denied by user"` (line 180) and toast error `"Copy not available in this context"` (line 68). Neither is ornate; both are functional status messages. No tropes detected.
