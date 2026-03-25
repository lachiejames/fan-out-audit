# Tropes Audit: apps/app/src/pages/ai — Slice 106

Files audited (resolved actual paths, as flat paths no longer exist):

- `apps/app/src/pages/ai/components/knowledge-graph/MobileKnowledgeGraphFallback.tsx`
- `apps/app/src/pages/ai/components/learned-patterns/LearnedPatternsPanel.tsx`
- `apps/app/src/pages/ai/page.tsx`
- `apps/app/src/pages/ai/tabs/BehaviorTab.tsx`

Note: `ContentPicker.tsx` and `ContextSection.tsx` do not exist at the listed paths or anywhere under `apps/app/src/pages/ai/`. They were not found in the directory listing and could not be audited. The four files above are what exist and were read.

---

## MobileKnowledgeGraphFallback.tsx

No user-visible copy beyond structural labels ("Explore neighbors", "Load more", "Open in list") and category names (preference, fact, relationship, etc.). All labels are functional. No findings.

---

## LearnedPatternsPanel.tsx

**Finding 1 — section heading:** Line 65.

```tsx
<h3 className="text-sm font-medium text-foreground">Learned Writing Patterns</h3>
```

"Learned" is passive AI-magic framing. The patterns are extracted by the system from the user's edits. The label does not tell the user what these patterns represent or how they are used.

**Suggested fix:** "Writing Patterns" with a subtitle like "Extracted from your edits — used to shape draft tone."

**Finding 2 — cold-start / progress text:** The copy comes from utility functions (`getProgressText`, `getEmptyStateText`) not visible in this file, but the rendered output via `getDecayHelperText()` / `getDecayExpandedText()` is surfaced here. These cannot be audited without reading the utility file. Flag for follow-up audit of `apps/app/src/shared/utils/pattern-learning-notification.utils.ts` and `apps/app/src/pages/ai/components/learned-patterns/learned-patterns-display.utils.ts`.

---

## page.tsx

**Finding — page description copy:** Lines 86-88.

```tsx
<p className="text-sm text-zinc-400">Your AI's knowledge base and behavioral observations. Track what it's learned.</p>
```

"Your AI's knowledge base" is generic and slightly possessive-AI framing. "Track what it's learned" implies an autonomous learner. This is a minor instance but sits on the main page header, making it high-visibility.

**Suggested fix:** "What SlopWeaver knows about you and how you work." — more direct, drops the "tracking an AI" framing.

---

## BehaviorTab.tsx

**Finding 1 — paywall card description:** Lines 60-63.

```tsx
<h3 className="font-semibold text-lg text-zinc-100">Behavioral Fingerprint</h3>
<p className="mt-1 max-w-md text-sm text-zinc-400">
  See how your AI understands your work patterns. Response time trends, formality shifts, and communication
  style across channels.
</p>
```

"See how your AI understands your work patterns" is AI-magic framing — "understands" implies comprehension rather than measurement. The remainder of the sentence is specific and good (response times, formality shifts, channels). The lead clause weakens it.

**Suggested fix:** "See how you communicate: response time trends, formality shifts, and tone by channel."

**Finding 2 — cold-start card copy:** Lines 83-86.

```tsx
<h3 className="font-semibold text-lg text-zinc-100">Building your profile</h3>
<p className="mt-1 max-w-md text-sm text-zinc-400">
  Your behavioral profile will appear after enough messages are analyzed. The analysis runs nightly.
</p>
```

"Building your profile" and "enough messages are analyzed" are vague. The user cannot know what "enough" means or take any action. "Building" implies active construction happening now, which may not be accurate.

**Suggested fix:** "Not enough data yet. Behavioral analysis runs nightly once [N] messages have been synced." If the threshold is not exposed, drop the qualifier: "Behavioral analysis runs nightly. Check back after a few days of use."

**Finding 3 — cold-start bullet list item:** Line 93.

```tsx
<li>Active hours and peak productivity</li>
```

"Peak productivity" is a marketing-adjacent phrase. The tab measures response time and active hours, not productivity in any meaningful sense.

**Suggested fix:** "Active hours and busiest time of day."

**Finding 4 — behavior content description:** Lines 421-424.

```tsx
<p className="text-sm text-zinc-400">
  How your AI understands your work patterns. These signals help drafts and prioritization feel more like you.
</p>
```

"How your AI understands your work patterns" repeats the same trope from the paywall card. "feel more like you" is soft AI-magic copy.

**Suggested fix:** "Measured signals from your messages — used to tune draft tone and prioritization."

---

## Summary

| File                             | Findings                                                                  |
| -------------------------------- | ------------------------------------------------------------------------- |
| MobileKnowledgeGraphFallback.tsx | None                                                                      |
| LearnedPatternsPanel.tsx         | 1 copy finding + 1 flag for utility file follow-up                        |
| page.tsx                         | 1 — page header "Track what it's learned" framing                         |
| BehaviorTab.tsx                  | 4 — paywall card, cold-start card (2 instances), live content description |
| ContentPicker.tsx                | File not found — cannot audit                                             |
| ContextSection.tsx               | File not found — cannot audit                                             |
