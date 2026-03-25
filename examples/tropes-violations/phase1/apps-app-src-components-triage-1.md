# AI Writing Tropes Audit — apps/app/src/components/triage (slice 1)

**Files audited** (8 requested; 4 exist, 4 not found):

| File                        | Status            |
| --------------------------- | ----------------- |
| `IdleWarningBanner.tsx`     | Exists — audited  |
| `TriageStatsPanel.tsx`      | Exists — audited  |
| `TriageTriggerBanner.tsx`   | Exists — audited  |
| `TriageTriggerProvider.tsx` | Exists — audited  |
| `triage-card.tsx`           | Not found in repo |
| `triage-queue-controls.tsx` | Not found in repo |
| `triage-setup-view.tsx`     | Not found in repo |
| `triage-slide-over.tsx`     | Not found in repo |

---

## IdleWarningBanner.tsx

**Line 45:**

```tsx
Session will pause in <span>…{secondsRemaining}s</span> — tap to stay active
```

**Trope: em dash used as a stylistic pause.**
The em dash in "— tap to stay active" is a typical AI writing tic. Replace with a comma or restructure the sentence.

Suggested fix: "Session pauses in {secondsRemaining}s. Tap to stay active."

---

## TriageStatsPanel.tsx

**Line 40:**

```tsx
<p className="text-center text-muted-foreground">Complete your first session to see stats!</p>
```

**Trope: Enthusiastic exclamation mark on a hollow call-to-action.**
"Complete your first session to see stats!" reads like AI-generated filler encouragement. It also tells the user something they already know (the stats panel is empty because they haven't triaged yet). Remove the exclamation mark and rewrite to something concrete.

Suggested fix: "Your stats appear here after your first session."

---

## TriageTriggerBanner.tsx

**Lines 71-72:**

```tsx
const headline = "Backlog detected";
const subline = `${trigger.unreadCount} unread messages. Triage them now?`;
```

**Trope: "Detected" as AI system language for a simple count threshold.**
"Backlog detected" is robotic and clinical. A threshold of 25+ unread messages being called a "detected" backlog sounds like a machine reporting a fault condition. Use plain language.

Suggested fix for headline: "Your inbox is piling up" or simply "{trigger.unreadCount} unread messages".

**Trope: Rhetorical question ("Triage them now?") as a call-to-action.**
The question format is a soft AI writing pattern. The button already says "Start triage" — the subline question is redundant and slightly nagging. Replace with a statement.

Suggested fix for subline: `${trigger.unreadCount} unread messages` (let the button do the asking).

---

## TriageTriggerProvider.tsx

No violations found.

---

## Files Not Found

`triage-card.tsx`, `triage-queue-controls.tsx`, `triage-setup-view.tsx`, and `triage-slide-over.tsx` do not exist at the listed paths. They were not audited.
