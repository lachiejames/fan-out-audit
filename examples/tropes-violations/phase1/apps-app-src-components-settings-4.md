# AI Writing Tropes Audit - Settings Components Slice 98

**Files audited** (8):

- `apps/app/src/components/settings/settings-knowledge-sources.tsx`
- `apps/app/src/components/settings/settings-learned-rules-panel.tsx`
- `apps/app/src/components/settings/settings-model-controls-card.tsx`
- `apps/app/src/components/settings/settings-planned-badge.tsx`
- `apps/app/src/components/settings/settings-planned-placeholder.tsx`
- `apps/app/src/components/settings/settings-platform-usage.tsx`
- `apps/app/src/components/settings/settings-rules.tsx`
- `apps/app/src/components/settings/settings-search.tsx`

---

## Findings

### 1. `settings-rules.tsx` line 118

**Violation: Vague filler description**

```
Define deterministic rules that guide triage, replies, and automations. These rules are always respected.
```

"These rules are always respected" is the AI-safety reassurance formula applied to a settings page. It reads as defensive rather than informative. The word "always" is doing no work - of course user-defined rules are respected, that is the point of the feature.

**Fix:** Drop the second sentence. The first sentence is sufficient and concrete.

---

### 2. `settings-learned-rules-panel.tsx` line 62

**Violation: Passive "nothing yet" empty state that buries the mechanic**

```
When you consistently correct AI suggestions during triage, rules are learned automatically.
```

"Automatically" is a filler qualifier here. More importantly, "consistently correct" is vague about what threshold triggers learning. This is a minor trope instance - the passive framing ("are learned automatically") obscures who learns what from what.

**Fix:** "Correct the same kind of suggestion a few times and SlopWeaver locks it in as a rule." More concrete, active voice, gives a mental model.

---

### 3. `settings-search.tsx` line 86 (searchable item description)

**Violation: "Deterministic" jargon in user-facing copy**

```
description: "Define deterministic AI rules",
```

"Deterministic" is an engineering term that means nothing to most users. This appears in the search results panel where a non-technical user might encounter it.

**Fix:** "Set rules the AI always follows" or "Define fixed rules for triage and replies."

---

## Summary

| File                               | Line | Trope                                                     |
| ---------------------------------- | ---- | --------------------------------------------------------- |
| `settings-rules.tsx`               | 118  | "always respected" AI-safety filler sentence              |
| `settings-learned-rules-panel.tsx` | 62   | Passive "are learned automatically" obscures the mechanic |
| `settings-search.tsx`              | 86   | "deterministic" jargon in user-facing search result copy  |

`settings-knowledge-sources.tsx`, `settings-model-controls-card.tsx`, `settings-planned-badge.tsx`, `settings-planned-placeholder.tsx`, and `settings-platform-usage.tsx` contain no AI writing tropes. Their copy is either functional UI labels or absent (skeleton/badge components).
