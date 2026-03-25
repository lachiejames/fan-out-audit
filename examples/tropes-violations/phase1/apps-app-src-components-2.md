# Tropes Audit: apps/app/src/components (batch 2)

Files audited:

- `ai-chat-tool-sources-summary.tsx`
- `ai-chat-widget-empty-state.tsx`
- `ai-chat-widget-file-preview.tsx`
- `ai-chat-widget-fullscreen.tsx`
- `ai-chat-widget-panel.tsx`
- `ai-chat-widget-voice-controls.tsx`
- `ai-chat-widget.composer.tsx`
- `ai-chat-widget.messages.tsx`

---

## Findings

### 1. Vague aspirational subtitle (serves-as dodge / ornate noun)

**File**: `ai-chat-widget-fullscreen.tsx`, line 162
**Text**: `"The AI that learns how you work"`
**Trope**: Serves-as dodge. This is the product's tagline used as a subtitle inside the chat header. It makes a claim without saying what the AI actually does at this moment. The companion product tagline ("The AI that proves it's learning you") is more concrete; this variant drops the accountability word "proves" and regresses to a vague promise.
**Fix**: Remove the subtitle entirely, show current model or a concrete status, or use the canonical tagline if the tagline must appear.

---

### 2. Pedagogical / coaching tone in empty state

**File**: `ai-chat-widget-empty-state.tsx`, lines 21-22
**Text**:

```
"I'm still learning your patterns."
"Ask me anything, or track my improvement in Analytics."
```

**Trope**: Pedagogical voice. The copy addresses the user in a coaching register ("Ask me anything") and self-narrates the AI's developmental journey. "I'm still learning your patterns" is vague reassurance rather than useful information. "Track my improvement" is stakes inflation masquerading as a feature prompt.
**Fix**: Say something concrete about what the user can do right now. Example: "Ask a question or pick one below." Move the Analytics prompt elsewhere, or make it more direct: "See your acceptance rate in Analytics."

---

### 3. Slow-stream fallback copy (false suspense)

**File**: `ai-chat-widget.messages.tsx`, line 197
**Text**: `"Still working on this..."`
**Trope**: False suspense. The ellipsis and phrasing "still working on this" dramatises latency. It sounds like the AI is deliberating rather than simply processing.
**Fix**: Use a neutral status. Example: "Processing..." or no label at all (the typing indicator is already visible).

---

### 4. "Loading..." transient label in audio player

**File**: `ai-chat-widget-file-preview.tsx`, line 97
**Text**: `"Loading..."`
**Trope**: Minor false suspense. The word is fine as a loading state, but the ellipsis adds unnecessary drama. This is a low-severity finding.
**Fix**: `"Loading"` (no ellipsis), or omit the label until duration is available.

---

## No findings for

- `ai-chat-tool-sources-summary.tsx` — functional label only ("Used N tools"), clean.
- `ai-chat-widget-file-preview.tsx` — file names and sizes only, no prose. (See minor note on "Loading..." above.)
- `ai-chat-widget-panel.tsx` — "Context active" and "On {page}" are concise and functional. "Drop files here" is direct.
- `ai-chat-widget-fullscreen.tsx` (drag overlay) — "Drop files here" and the file-type list are plain and correct.
- `ai-chat-widget-voice-controls.tsx` — "Cancel recording" is functional, no tropes.
- `ai-chat-widget.composer.tsx` — "What do you want to do?" is direct. Voice state labels (Listening, Transcribing, Synthesizing, Speaking, Voice unavailable) are accurate and non-ornate.
- `ai-chat-widget.messages.tsx` (general) — No other prose in JSX beyond the slow-stream label noted above.
