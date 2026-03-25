# Tropes Audit: apps/app/src/components/platform-views (slice 1)

Files audited:

- `asana-task-view.tsx`
- `chat-thread-view.tsx`
- `facebook-messenger-view.tsx`
- `github-comment-item.tsx`
- `github-file-change-item.tsx`
- `github-label-badge.tsx`
- `github-review-item.tsx`
- `github-user-avatar.tsx`

## Findings

### 1. "Unknown" as author fallback (multiple files)

**Files**:

- `asana-task-view.tsx`, line 108: `comment.author?.name ?? "Unknown"`
- `github-comment-item.tsx`, line 24: `comment.author?.login ?? "Unknown"`
- `github-review-item.tsx`, line 36: `review.author?.login ?? "Unknown"`

**Category**: Generic placeholder copy

**Problem**: "Unknown" is a developer-facing null-state label, not a considered user-facing string. It surfaces to the user in an author byline context where it looks like an actual display name. It is also redundant - if there is no author, the UI could simply omit the name or show nothing.

**Suggested fix**: Either omit the author name when null, or use a less label-like fallback such as "Deleted user" or simply an empty string that collapses gracefully with the surrounding layout.

---

### 2. "[No text content]"

**File**: `facebook-messenger-view.tsx`, line 40
**Category**: Debug/developer leak in user-facing UI

**Text**:

```
message.message || "[No text content]"
```

**Problem**: Square-bracketed strings in the style of `[placeholder]` are a developer convention for marking absent values, not user copy. A user receiving a Messenger message with no text body (e.g., a sticker or attachment-only message) would see the literal string `[No text content]` in a message bubble, which reads as a system artifact rather than useful information.

**Suggested fix**: Render nothing, a subtle italic label like "Attachment" or "Unsupported message type", or omit the bubble entirely for messages with no text body.

---

### 3. "No messages"

**File**: `facebook-messenger-view.tsx`, line 160
**Category**: Terse empty-state copy

**Text**:

```
"No messages"
```

**Problem**: Bare factual statement with no context. It is technically accurate but offers nothing to orient the user. In a Messenger thread view that should always have messages, this state likely indicates a load failure or data gap, not a genuinely empty thread.

**Suggested fix**: "No messages loaded" or simply remove the empty state if a thread with zero messages is not a valid product state.

---

### 4. "Write a comment... (Cmd+Enter to post)"

**File**: `asana-task-view.tsx`, line 151
**Category**: Minor - keyboard shortcut parenthetical in placeholder

**Text**:

```
placeholder="Write a comment... (Cmd+Enter to post)"
```

**Problem**: Embedding `(Cmd+Enter to post)` inside the placeholder text is a common but clunky pattern. Placeholder text disappears the moment the user starts typing, so the shortcut hint vanishes exactly when it would be useful. It also uses a Mac-specific modifier name unconditionally.

**Suggested fix**: Surface the keyboard shortcut as persistent helper text below the textarea, or as a tooltip on the send button, rather than inside the placeholder.

---

No other AI writing tropes found. `chat-thread-view.tsx`, `github-file-change-item.tsx`, `github-label-badge.tsx`, and `github-user-avatar.tsx` contain no user-facing copy beyond structural labels.
