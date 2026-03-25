# Tropes Audit: apps/app/src/pages/contacts

## Files Audited

- `apps/app/src/pages/contacts/components/EntityGraphView.tsx` (actual path; listed path `contacts/EntityGraphView.tsx` does not exist)
- `apps/app/src/pages/contacts/loading.tsx`
- `apps/app/src/pages/contacts/page.tsx`

## Findings

### `components/EntityGraphView.tsx`

**1 violation** - AI trope in empty state copy

File: `apps/app/src/pages/contacts/components/EntityGraphView.tsx`, line 325

```tsx
<p className="mt-1 text-xs text-zinc-600">Connect integrations to see your contact network grow.</p>
```

"See your contact network grow" is a soft AI-product trope: it promises a future emergent outcome in order to motivate action. The phrasing is vague and slightly breathless. The empty state already has a clear action ("No contacts to display in the entity network."). The second line adds nothing concrete.

Suggested fix: Remove the second line, or replace with something factual: "Contacts appear once you connect an integration and sync completes."

### `loading.tsx`

No findings. Pure skeleton UI, no user-facing copy.

### `page.tsx`

No findings. All inline strings are UI labels or navigation prompts with no AI writing tropes.
