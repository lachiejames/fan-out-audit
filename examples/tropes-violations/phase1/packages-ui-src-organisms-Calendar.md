# Slice 192: packages/ui/src/organisms/Calendar

Files audited:

- `packages/ui/src/organisms/Calendar/MeetingCard.tsx`
- `packages/ui/src/organisms/Calendar/types.ts`

---

## MeetingCard.tsx

**User-facing strings:**

- `"AI has context"` — shown inline on meeting cards when `meeting.hasContext` is true.
- `"Attendees"` — section label.
- `"related messages from attendees"` — e.g. "3 related messages from attendees".
- `"Action items from last meeting"` — section label.
- `"Join Google Meet"` / `"Join Zoom"` / `"Join Meeting"` — join buttons.

### Violation: "AI has context"

**File:** `packages/ui/src/organisms/Calendar/MeetingCard.tsx`, line 50

```tsx
<span>AI has context</span>
```

**Trope:** Vague AI capability claim. "Context" is undefined from the user's perspective. Does it mean it read related emails? Knows the attendees? Has meeting notes? The string is technically true but reveals nothing actionable.

**Severity:** Low-medium. It appears on meeting cards and may be a meaningful signal for users, but the phrasing is the classic "AI knows things" hand-wave. Should be replaced with something specific, e.g. "Has related emails" or the actual count/type of context.

---

## types.ts

Type definitions only. No user-facing strings. No violations.

---

## Findings

| File                       | Line | String             | Trope                     | Severity   |
| -------------------------- | ---- | ------------------ | ------------------------- | ---------- |
| `Calendar/MeetingCard.tsx` | 50   | `"AI has context"` | Vague AI capability claim | Low-medium |
