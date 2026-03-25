# Tropes Audit: apps/app/src/components (batch 3)

Files audited:

- `ai-chat-widget.tsx`
- `ai-floating-logo.tsx`
- `ai-logo-chat.tsx`
- `ai-suggestion-chips.tsx`
- `ai-tool-action-button.tsx`
- `ai-tool-group.tsx`
- `ai-tool-row-approval-actions.tsx`
- `ai-tool-row.tsx`

---

## Findings

### 1. Suggestion chip: "Summarize my inbox"

**File**: `ai-chat-widget.tsx`, line 353
**Text**: `"Summarize my inbox"`
**Trope**: "Summarize" is a generic, overused AI capability label. Every AI assistant ships a "summarize" chip.
**Suggestion**: Replace with something specific to SlopWeaver's value prop, e.g. "What needs a reply today?" or "What did I miss while I was out?"

---

### 2. Tooltip: "Open assistant"

**File**: `ai-floating-logo.tsx`, line 53 (default case) and line 61 (`aria-label`)
**Text**: `"Open assistant"` / `aria-label="Open assistant"`
**Trope**: "Assistant" is generic AI-product filler. Every product calls their AI an "assistant."
**Suggestion**: Use a name or a concrete action. Options: "Open SlopWeaver", "Ask SlopWeaver", or keep the page-contextual variants ("Ask about inbox", "Ask about tasks") and make the default match that pattern, e.g. "Ask anything".

---

### 3. Tooltip: "Review queue"

**File**: `ai-floating-logo.tsx`, line 51
**Text**: `"Review queue"`
**Trope**: Not an AI writing trope per se, but the phrasing is ambiguous. Minor flag only.
**Suggestion**: "Open your queue" or "Check your queue" is clearer about who acts.

---

### 4. Running indicator: "Searching..."

**File**: `ai-tool-group.tsx`, line 231 and 266
**Text**: `"Searching..."` / `"Searching"`
**Trope**: Both the spinner-only state and the platform-icon state use "Searching" with no object. This is the canonical AI loader trope alongside "Thinking..." and "Generating...".
**Suggestion**: Add specificity: "Searching your apps" or "Looking through Gmail, Slack..." (the platform list is already rendered next to the label in the platforms state, so the word "Searching" alone adds nothing). In the spinner-only state (before platforms are known), "Checking your tools" or simply removing the label and relying on the spinner is cleaner.

---

### 5. Approval prompt: "Approve this action?"

**File**: `ai-tool-row-approval-actions.tsx`, line 102
**Text**: `"Approve this action?"`
**Trope**: "Action" is a placeholder word. "This action" tells the user nothing about what they are approving.
**Suggestion**: The tool name is rendered directly above this prompt (in `ApprovalToolRow` in `ai-tool-group.tsx`), so the prompt can drop the object entirely: "Go ahead?" / "Run it?" / just "Approve?" with the tool label as context. Alternatively surface the action's effect: "Send this reply?" when the tool is a send action.

---

### 6. Status label: "Needs approval"

**File**: `ai-tool-group.tsx`, line 463
**Text**: `"Needs approval"`
**Trope**: Bureaucratic phrasing. Minor flag. This is a status badge next to a tool name.
**Suggestion**: "Waiting for you" or "Your call" is friendlier and matches SlopWeaver's voice better than corporate approval-workflow language.

---

### 7. Toast copy: "Reasoning enabled" / "Reasoning disabled"

**File**: `ai-chat-widget.tsx`, lines 196-197
**Text**: `"Reasoning enabled"` / `"Reasoning disabled"`
**Trope**: These are feature-toggle confirmations using internal model terminology ("Reasoning") that a user may not map to what the product actually does.
**Suggestion**: "Deep thinking on" / "Deep thinking off" or "Extended reasoning on/off" matches the UI label ("Deep", "Normal", "Light") shown nearby, and sounds like product copy rather than a model capability flag.

---

## No-finding notes

- `ai-logo-chat.tsx`: No user-facing text.
- `ai-suggestion-chips.tsx`: The chip labels are passed in as props; no hardcoded text in this file beyond icon mappings.
- `ai-tool-action-button.tsx`: The button label comes from `action.label` (a prop). No hardcoded user-facing copy other than error toasts ("Could not start reauthorization. Please try again shortly." and "Could not start OAuth flow. Please try again.") which are functional error messages, not AI tropes.
- `ai-tool-row.tsx`: User-facing text is "Action denied by user" (line 64) which is accurate status copy, not a trope.
