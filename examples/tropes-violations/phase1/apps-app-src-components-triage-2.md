# AI Writing Tropes Audit — apps/app/src/components/triage (slice 2)

**Files audited** (2):

- `triage-stat-card.tsx`
- `triage-view.tsx`

---

## triage-stat-card.tsx

No violations found. The component renders label/value pairs passed in as props with no user-visible copy of its own.

---

## triage-view.tsx

**Line 48:**

```tsx
<span className="font-semibold text-white tabular-nums">{pendingItems.length}</span>
<span className="hidden xs:inline"> items</span> remaining
```

No violation here — "remaining" is plain and accurate.

**Line 154:**

```tsx
<h3 className="mb-2 text-[10px] font-semibold uppercase tracking-wide text-zinc-500">Shortcuts</h3>
```

No violation — "Shortcuts" is a plain label.

**Line 176:**

```tsx
<h3 className="mb-2 text-[10px] font-semibold uppercase tracking-wide text-zinc-500">Classification</h3>
```

No violation — "Classification" is a plain label.

**Lines 194-196:**

```tsx
<span className="text-zinc-500">AI classified</span>
```

**Trope: "AI classified" as a label for a metric bar.**
The sidebar stat row labels the tier-1 categorization count as "AI classified." This is fine as a technical label inside an in-session sidebar that the user has already opted into. Borderline — not flagged as a required fix, but worth considering whether "Predicted" or "Auto-sorted" is clearer to non-technical users.

No hard violations found.
