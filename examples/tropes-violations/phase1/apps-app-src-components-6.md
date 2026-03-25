# Tropes Audit: apps/app/src/components (batch 6)

Files audited:

- `apps/app/src/components/knowledge-edit-modal.tsx`
- `apps/app/src/components/markdown-content.tsx`
- `apps/app/src/components/message-slide-over-create-task-dialog.tsx`
- `apps/app/src/components/message-slide-over-header.tsx`
- `apps/app/src/components/message-slide-over-platform-loader.tsx`
- `apps/app/src/components/message-slide-over-slack-actions.tsx`
- `apps/app/src/components/message-slide-over.tsx`
- `apps/app/src/components/navigation-progress.tsx`

---

## Findings

### 1. `message-slide-over.tsx` — line 206: "Opening reply composer with predicted text"

```tsx
toast.info("Opening reply composer with predicted text");
```

**Trope**: Narrating system actions back to the user. The toast describes what the app is doing in implementation terms ("reply composer", "predicted text") rather than confirming an outcome the user cares about. The user clicked "Edit" on a prediction — they already know what is about to happen.

**Fix**: Remove the toast entirely. The UI transition (prediction panel closing, composer opening) is sufficient feedback. If a toast is needed, make it outcome-oriented: nothing here warrants one.

---

### 2. `message-slide-over.tsx` — line 220: "New prediction generated"

```tsx
toast.success("New prediction generated");
```

**Trope**: Passive-voice system jargon. "Prediction generated" frames the action from the machine's perspective using internal terminology ("prediction"). Users do not know or care that the system has a concept called a "prediction" — they asked SlopWeaver to suggest a reply.

**Fix**: Use user-outcome language. For example: "New reply suggestion ready" or simply "Done — tap to review."

---

### 3. `message-slide-over-create-task-dialog.tsx` — line 122: "Turn this message into a task in your task list."

```tsx
<DialogDescription>Turn this message into a task in your task list.</DialogDescription>
```

**Trope**: Redundant description that restates what the dialog title already says ("Create Task") without adding information. "Your task list" is also slightly padded — users know it is their task list.

**Fix**: Either remove the description entirely (the title and input placeholder are self-explanatory) or make it earn its place with a concrete, useful detail: e.g., "Saved to Tasks. You can edit it there."

---

### 4. `knowledge-edit-modal.tsx` — line 79: placeholder "Enter knowledge content..."

```tsx
placeholder = "Enter knowledge content...";
```

**Trope**: Internal jargon exposed to users. "Knowledge content" is system/developer language. Users editing an entry in their knowledge base do not naturally think of it as "knowledge content."

**Fix**: Replace with something concrete: "What do you want SlopWeaver to remember?" or simply "Add a note about yourself, your preferences, or your work."

---

## No-finding files

- `markdown-content.tsx` — no user-facing strings; purely a rendering utility.
- `message-slide-over-header.tsx` — user-facing strings are minimal ("Back", "Close message") and plain.
- `message-slide-over-platform-loader.tsx` — no user-facing strings.
- `message-slide-over-slack-actions.tsx` — error strings are developer-/internal-facing toasts surfacing raw technical reasons ("Missing integrationId", "Missing platform identifiers"). These are error states that should arguably be improved, but they are not AI writing tropes — they are straightforward (if terse) error messages.
- `navigation-progress.tsx` — no user-facing strings.
