# Tropes Audit: apps/app/src/components/settings (batch 2)

Files audited:

- `components/settings/knowledge-sources/searchable-archive-section.tsx`
- `components/settings/profile/settings-ai-context-modal.tsx`
- `components/settings/profile/settings-ai-snapshot-card.tsx`
- `components/settings/profile/settings-basics-overrides-section.tsx`
- `components/settings/profile/settings-current-focus-section.tsx`
- `components/settings/profile/settings-people-section.tsx`
- `components/settings/profile/settings-profile-header.tsx`
- `components/settings/profile/settings-voice-section.tsx`

---

## Findings

### 1. `settings-ai-snapshot-card.tsx` — line 208: "learned"

**Location**: `settings-ai-snapshot-card.tsx`, line 208

**Offending text**:

```tsx
{importedKnowledgeCount} items learned
```

**Why it's a trope**: "Learned" is an AI anthropomorphism cliche. It frames the app as a conscious student accumulating wisdom. The actual behavior is importing and indexing knowledge source items. This is also imprecise: the count refers to items imported from knowledge sources, not behavioral patterns learned from usage.

**Fix**: Replace with something factual, e.g. `{importedKnowledgeCount} items imported` or `{importedKnowledgeCount} knowledge items`.

---

### 2. `settings-ai-context-modal.tsx` — line 133: "What the AI Knows"

**Location**: `settings-ai-context-modal.tsx`, line 133

**Offending text**:

```tsx
<DialogTitle>What the AI Knows</DialogTitle>
```

**Why it's a trope**: "Knows" anthropomorphizes the AI. The dialog shows the serialized profile context that gets injected into prompts. It does not show "knowledge" in any cognitive sense. The subtitle on line 134 is accurate ("This profile is included in every AI prompt") but the title contradicts it with fuzzy AI-knows framing.

**Fix**: Use the subtitle's accurate framing instead, e.g. "AI Profile Context" or "What's in Your AI Profile".

---

### 3. `settings-voice-section.tsx` — line 118: "Still learning your style"

**Location**: `settings-voice-section.tsx`, line 118-119

**Offending text**:

```tsx
Still learning your style. Keep sending messages and SlopWeaver will pick up your patterns.
```

**Why it's a trope**: Two tropes in two sentences. "Still learning" is the classic AI-as-eager-student framing. "Pick up your patterns" is vague and anthropomorphizes pattern extraction as casual human observation. Both phrases obscure what is actually happening: the AI has not yet extracted enough signal from the user's communication history to populate the voiceprint fields.

**Fix**: Be direct about the actual state, e.g. "Not enough message history yet. Send more messages to populate your voiceprint." or simply "Voiceprint will fill in once more messages are processed."

---

### 4. `settings-ai-snapshot-card.tsx` — line 107: "Not enough data for a profile yet. It fills in as you use your connected tools."

**Location**: `settings-ai-snapshot-card.tsx`, line 107

**Offending text**:

```tsx
return "Not enough data for a profile yet. It fills in as you use your connected tools.";
```

**Why it's a trope**: "Fills in" is vague and implies an organic, autonomous growth process. The phrasing is soft AI-magic language. The actual mechanism is explicit: the profile is computed from synced messages and calendar events. Users deserve to understand the mechanism.

**Fix**: e.g. "Profile builds from your synced messages and calendar. Connect more tools to see data here." This is more concrete and also serves as a call to action.

---

### 5. `settings-current-focus-section.tsx` — line 54: "SlopWeaver will surface your focus areas as you work."

**Location**: `settings-current-focus-section.tsx`, line 54

**Offending text**:

```tsx
<p className="text-muted-foreground text-sm">SlopWeaver will surface your focus areas as you work.</p>
```

**Why it's a trope**: "Surface" is overused AI product jargon. Combined with "as you work" it implies ambient AI awareness watching the user. The actual mechanism is periodic profile updates from synced content.

**Fix**: e.g. "Projects and topics appear here once messages and calendar events are synced." Concrete, no magic.

---

### 6. `settings-people-section.tsx` — line 22: "high-context relationships"

**Location**: `settings-people-section.tsx`, line 22

**Offending text**:

```tsx
<p className="text-muted-foreground text-sm">The people SlopWeaver treats as high-context relationships.</p>
```

**Why it's a trope**: "High-context relationships" is AI-product jargon that sounds meaningful but explains nothing. Users won't know what "high-context" means in this context or how the list is built.

**Fix**: e.g. "People you communicate with most across connected tools." or "Frequent contacts detected from your messages and calendar."

---

### 7. `settings-people-section.tsx` — line 29-31: "important people will show up here"

**Location**: `settings-people-section.tsx`, lines 29-31

**Offending text**:

```tsx
<p className="text-muted-foreground text-sm">As you work across tools, important people will show up here.</p>
```

**Why it's a trope**: "Show up here" and "as you work" repeat the ambient-AI-awareness pattern from the focus section. "Important people" is vague — what makes someone important? Frequency? Recency? The AI deciding?

**Fix**: e.g. "Frequent contacts appear here once messages are synced." Explains the mechanism, removes the magic.

---

### 8. `settings-profile-header.tsx` — line 33: "Your AI Work Profile"

**Location**: `settings-profile-header.tsx`, line 33

**Offending text**:

```tsx
<h2 className="font-bold text-2xl text-foreground">Your AI Work Profile</h2>
```

**Why it's a trope**: "AI Work Profile" is a redundant label (everything in the product is AI-assisted). The page already lives under Settings > Profile, so users have context. The "AI" qualifier does no work here and reads as buzzword padding.

**Fix**: "Work Profile" or simply "Profile" — the AI angle is implicit everywhere in the product. Alternatively a more specific title if the intent is to distinguish from a general user profile: "SlopWeaver Profile" or "Learned Profile".

---

## Summary

| File                                 | Line(s) | Violation                                                    |
| ------------------------------------ | ------- | ------------------------------------------------------------ |
| `settings-ai-snapshot-card.tsx`      | 208     | "items learned" — anthropomorphism for indexing              |
| `settings-ai-context-modal.tsx`      | 133     | "What the AI Knows" — cognitive anthropomorphism             |
| `settings-voice-section.tsx`         | 118-119 | "Still learning your style" / "pick up your patterns"        |
| `settings-ai-snapshot-card.tsx`      | 107     | "fills in as you use" — vague AI magic                       |
| `settings-current-focus-section.tsx` | 54      | "surface your focus areas as you work" — AI-awareness jargon |
| `settings-people-section.tsx`        | 22      | "high-context relationships" — unexplained jargon            |
| `settings-people-section.tsx`        | 29-31   | "important people will show up here" — ambient AI framing    |
| `settings-profile-header.tsx`        | 33      | "Your AI Work Profile" — buzzword padding                    |
