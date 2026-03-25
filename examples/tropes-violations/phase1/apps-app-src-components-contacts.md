# Tropes Audit: apps/app/src/components/contacts

Files audited:

- contact-card.tsx
- contact-detail-panel.tsx
- contacts-empty-state.tsx
- contacts-header.tsx
- merge-suggestion-card.tsx
- vip-section.tsx

---

## Findings

### 1. "Notes" section header uses a Circle (orb) icon for AI-derived content — contact-detail-panel.tsx, line 231

```tsx
<Circle className="h-4 w-4 text-primary" />;
Notes;
```

The section is titled "Notes" but its content comes from `contact.aiKnowledge` (an AI-extracted array of facts). The Circle icon (the SlopWeaver AI orb) is used as a visual marker, but the section heading says "Notes" rather than communicating that these are AI-observed facts. This is a mild but meaningful mislabeling: users writing their own notes expect to edit them; AI-extracted knowledge is read-only and has different provenance. The mismatch buries the product's core value ("the AI that proves it's learning you") behind a generic word.

**Suggested fix:** Rename the heading to "What SlopWeaver knows" or "Observed facts" and keep the orb icon. This makes the AI origin explicit and lets the feature speak for itself. The editable notes section immediately below (also titled "Notes") should remain unchanged.

---

### 2. "SlopWeaver builds your contact graph as it learns who you interact with across tools." — contacts-empty-state.tsx, line 70

```tsx
description:
  "SlopWeaver builds your contact graph as it learns who you interact with across tools. Connect platforms to get started.",
```

"Learns" here is doing AI-hype work. The phrasing is accurate at a product level but sounds like a feature pitch rather than a helpful empty-state explanation. More practically, it does not tell the user what to do next or what they will see once contacts appear. The second sentence ("Connect platforms to get started.") is better but is tacked on after the marketing sentence.

**Suggested fix:** Lead with the outcome, not the mechanism: "Connect your tools and SlopWeaver will build your contact list automatically as messages come in." Removes "learns" without losing the meaning, and leads with what the user gets.

---

### 3. "mark important contacts as VIPs and their messages will get priority treatment" — contacts-empty-state.tsx, line 105

```tsx
description: "Mark important contacts as VIPs and their messages will get priority treatment",
```

"Priority treatment" is vague marketing language. It does not explain concretely what happens (messages are highlighted, shown first, or something else). This is a minor clarity issue rather than an AI trope, but it contributes to the pattern of abstract promises.

**Suggested fix:** "Mark important contacts as VIPs and their messages will be highlighted and shown first in your inbox." Concrete over vague.

---

### 4. "You'll be notified when duplicates are detected." — contacts-empty-state.tsx, line 171

```tsx
description: "No merge suggestions at the moment. You&apos;ll be notified when duplicates are detected.",
```

"Detected" is passive and machine-sounding. It also raises the question: detected by whom, and how? For a product whose positioning is "the AI that proves it's learning you," this is an opportunity to be specific and confident rather than vague.

**Suggested fix:** "No duplicates found right now. SlopWeaver will flag them as it sees more of your contacts." Warmer and more grounded.

---

## No findings

- contact-card.tsx: labels are functional (VIP, Merge suggested, platform names). No AI trope language.
- contacts-header.tsx: UI controls only. The subtitle "contacts across Gmail, Slack, and Linear" is accurate and specific (though it hardcodes three platforms rather than being dynamic, which is a separate code quality issue).
- merge-suggestion-card.tsx: "Same person?" is direct and functional. Confidence badges show percentages, which is concrete. No trope language.
- vip-section.tsx: "Your VIPs" is plain and appropriate. No trope language.
