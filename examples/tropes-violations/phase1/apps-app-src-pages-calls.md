# Tropes Audit: apps/app/src/pages/calls

## Files Audited

- `apps/app/src/pages/calls/call-card.tsx`
- `apps/app/src/pages/calls/call-slide-over.tsx`
- `apps/app/src/pages/calls/calls-empty-state.tsx`
- `apps/app/src/pages/calls/calls-error-state.tsx`
- `apps/app/src/pages/calls/page.tsx`

Note: `calls-header.tsx` does not exist at the listed path and was not audited.

## Findings

### `call-card.tsx`

No findings.

### `call-slide-over.tsx`

No findings.

### `calls-empty-state.tsx`

**1 violation** - Filler explanation of what AI is doing

File: `apps/app/src/pages/calls/calls-empty-state.tsx`, line 39

```tsx
Meeting transcripts will appear here once synced. SlopWeaver uses call context to improve its understanding of
your work.
```

The second sentence ("SlopWeaver uses call context to improve its understanding of your work") explains the AI's learning process in the empty state. This is an AI trope: filling silence with a promise about how the AI learns. The user already knows why they connected call integrations. The line is filler and reads like an AI product justifying its own existence.

Suggested fix: Drop the second sentence entirely. "Meeting transcripts will appear here once synced." is sufficient. Alternatively, say something concrete: "Your meeting transcripts are synced from Slack Huddles and other connected providers."

### `calls-error-state.tsx`

No findings.

### `page.tsx`

No findings. The inline header copy ("Meeting transcripts and recordings") is a plain descriptor with no tropes.
