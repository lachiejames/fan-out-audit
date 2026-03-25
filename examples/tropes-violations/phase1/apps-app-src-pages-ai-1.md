# Tropes Audit: apps/app/src/pages/ai — Slice 105

Files audited (resolved actual paths, as flat paths no longer exist):

- `apps/app/src/pages/ai/components/CoreProfileSection.tsx`
- `apps/app/src/pages/ai/components/FactsList.tsx`
- `apps/app/src/pages/ai/components/PlatformStyleSliders.tsx`
- `apps/app/src/pages/ai/components/TabLoadingState.tsx`
- `apps/app/src/pages/ai/tabs/KnowledgeTab.tsx` (covers ai-chat-section, knowledge-graph-section scope)
- `apps/app/src/pages/ai/tabs/ObservationsTab.tsx` (covers memories-tab, overview-tab scope)

Note: The originally listed flat paths (`ai-chat-section.tsx`, `knowledge-graph-section.tsx`, `memories-tab.tsx`, `overview-tab.tsx`) do not exist. The codebase has been reorganized into `components/` and `tabs/` subdirectories. The files above cover the same functional scope and are what was read and audited.

---

## CoreProfileSection.tsx

**Finding — empty-state copy:** Line 39-42.

```tsx
<p className="text-sm text-zinc-500 italic">
  No profile data yet. Start using SlopWeaver and it will build your profile.
</p>
```

"will build your profile" is vague AI-magic framing. The profile is constructed from specific signals (messages, edits, accepted drafts). The copy implies passive accumulation without telling the user what to do or what triggers it.

**Suggested fix:** "No profile yet. Accept or edit a few drafts and the profile will fill in."

---

## FactsList.tsx

No trope violations. The UI copy is functional and sparse: "Add something about yourself...", "Learned from: {source}", "View source", "Explore in knowledge graph". All phrasing is specific and action-oriented.

---

## PlatformStyleSliders.tsx

No trope violations. Labels are "Casual" / "Formal", "Auto-detected", "Custom", "Default" — all concrete and accurate.

---

## TabLoadingState.tsx

No copy present. Spinner only. No findings.

---

## KnowledgeTab.tsx (knowledge-graph-section / overview scope)

**Finding 1 — empty graph state copy:** Lines 241-245.

```tsx
<h3 className="mb-2 text-lg font-medium text-zinc-300">Your knowledge graph is growing</h3>
<p className="max-w-md text-sm text-zinc-500">
  As SlopWeaver learns from your messages and documents, it builds connections between facts, preferences,
  and relationships. Connect more integrations to accelerate graph growth.
</p>
```

"is growing" implies ongoing background activity when there may be nothing to show yet. "learns from" and "builds connections" are AI-magic verbs. "accelerate graph growth" reads like marketing copy inside a product screen.

**Suggested fix:** "No knowledge items yet. SlopWeaver extracts facts from synced messages and documents. Connect more integrations to give it more to work with."

**Finding 2 — "Learning Controls" section:** Lines 324-343.

```tsx
<span className="text-sm text-zinc-300">Learning from your messages</span>
```

"Learning from your messages" is a status indicator with no specificity. The pulsing green dot reinforces an always-on AI-magic impression. There is no way for the user to know what this actually means or whether it refers to a real in-progress operation.

**Suggested fix:** Either remove the indicator if it is always shown regardless of state, or make it conditional and specific: "Extracting patterns from recent messages" only when a job is actually running.

---

## ObservationsTab.tsx (memories-tab / overview scope)

**Finding — description copy:** Line 103.

```tsx
<p className="text-sm text-zinc-400">A time-ordered list of patterns and insights SlopWeaver detected.</p>
```

"patterns and insights" is a catch-all AI trope phrase. The actual items are proactive notifications (unanswered messages, hidden TODOs, threshold triggers). The copy does not reflect what users will actually see.

**Suggested fix:** "Proactive alerts from SlopWeaver — unanswered messages, found TODOs, and other flags." (Or similar, based on actual notification content.)

---

## Summary

| File                     | Findings                                                       |
| ------------------------ | -------------------------------------------------------------- |
| CoreProfileSection.tsx   | 1 — empty-state vague AI-magic copy                            |
| FactsList.tsx            | None                                                           |
| PlatformStyleSliders.tsx | None                                                           |
| TabLoadingState.tsx      | None                                                           |
| KnowledgeTab.tsx         | 2 — empty graph marketing copy; always-on "Learning" indicator |
| ObservationsTab.tsx      | 1 — "patterns and insights" catch-all description              |
