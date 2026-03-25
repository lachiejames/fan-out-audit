# Tropes Audit: apps/app/src/components/settings (batch 1)

Files audited:

- `data-security-card.tsx`
- `delete-account-modal.tsx`
- `knowledge-sources/knowledge-source-card.tsx`
- `knowledge-sources/knowledge-source-detail-dialog.tsx`
- `knowledge-sources/knowledge-source-imported-knowledge.tsx`
- `knowledge-sources/knowledge-source-upload-card.tsx`
- `knowledge-sources/knowledge-sources-empty-state.tsx`
- `knowledge-sources/quick-import-section.tsx`

---

## Findings

### 1. `knowledge-sources/knowledge-sources-empty-state.tsx` — line 21

**Text**: `"Bring your AI context"`

**Trope**: Filler heading. "Bring your X" is a soft, abstract imperative that says nothing concrete. The heading tells users what to do in the vaguest possible terms without stating any benefit.

**Fix**: Replace with a concrete description of what happens. Example: `"Start from your existing AI history"` or `"Import your ChatGPT and Claude history"`.

---

### 2. `knowledge-sources/knowledge-sources-empty-state.tsx` — line 22-25

**Text**: `"Upload ChatGPT or Claude exports to give SlopWeaver a head start on learning your patterns. Files, websites, or notes all work."`

**Trope**: "give X a head start on learning your patterns" is vague AI-flavored padding. "A head start" is a weasel phrase that avoids saying what actually happens. The sentence combines two distinct ideas (what to upload, what formats are supported) in a way that buries both.

**Fix**: State the concrete outcome. Example: `"Upload your ChatGPT or Claude export and SlopWeaver will extract your writing patterns, preferences, and working style. Supports files, websites, and manual notes."`

---

### 3. `knowledge-sources/quick-import-section.tsx` — line 103-104

**Text**: `"Extract knowledge items from this source and add them to your AI context."`

**Trope**: "AI context" is a jargon placeholder. Users don't think in terms of "AI context." This is internal implementation language dressed up as a description.

**Fix**: Describe what the user gets. Example: `"Extract facts, preferences, and patterns from this source so SlopWeaver can use them when drafting replies."` or more concisely: `"Pull facts and preferences out of this source and add them to what SlopWeaver knows about you."`

---

### 4. `knowledge-sources/quick-import-section.tsx` — line 134

**Text**: `"What SlopWeaver Learned"`

**Trope**: Anthropomorphization trope. Framing the system as "learning" like a student is an AI-startup cliche. It also raises questions about what "learned" means precisely.

**Fix**: Use a concrete label that describes the content. Example: `"Extracted Knowledge"` or `"Import Results"`. If the brand voice calls for something warmer: `"What we pulled from this source"`.

---

### 5. `knowledge-sources/quick-import-section.tsx` — lines 158-169 (top insights section)

**Text label**: `"Top insights"`

**Trope**: "Insights" is an overloaded AI buzzword. In context this label appears above a list of extracted facts/statements, which are not inherently "insights" in the analytical sense. The label inflates simple extracted text into something more impressive-sounding than it is.

**Fix**: `"Key extracts"`, `"Extracted items"`, or just remove the label and let the list speak for itself.

---

### 6. `data-security-card.tsx` — line 32

**Text**: `"Data encrypted & isolated"`

**Trope**: "isolated" is vague. Isolated from what? This is security reassurance copy that doesn't tell the user what is actually being protected or from whom.

**Fix**: Either make it precise (`"Encrypted, not shared with AI providers"`) or trim it to just the verifiable claim (`"End-to-end encrypted"`). The full card already lists the specifics, so the compact badge should state the most important single fact.

---

## Non-findings

- `delete-account-modal.tsx`: Copy is functional and direct. No tropes.
- `knowledge-source-card.tsx`: Purely data display, no descriptive prose. No tropes.
- `knowledge-source-detail-dialog.tsx`: Labels and section headings are factual (Parsed Summary, Revision History, Raw Data, Warnings). No tropes.
- `knowledge-source-imported-knowledge.tsx`: UI labels only (Approve Page, Reject Page, confidence %, category tags). No tropes.
- `knowledge-source-upload-card.tsx`: Form labels are concrete and accurate. Placeholder text (`"Paste or type your content here..."`, `"My meeting notes"`) is appropriate. No tropes.
