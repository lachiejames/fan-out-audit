# Tropes Audit: apps/app/src/components/command-palette

Files audited:

- command-palette-footer.tsx
- command-palette-partial-failure-warning.tsx
- command-palette-search-detail-panel.tsx
- command-palette.commands.tsx
- command-palette.tsx
- command-palette.utils.tsx

---

## Findings

### 1. "Interpreting your question..." — command-palette.tsx, line 317

```tsx
<span>Interpreting your question...</span>
```

Classic AI loading trope. The app detects when the query looks like a natural-language question (starts with what/how/why/etc.) and displays this badge while the user is still typing. It signals AI is "thinking" before any actual AI call has been made. The trigger is a regex on the query string, so this fires for any question-shaped text regardless of whether the assistant is doing anything. It is theatrical rather than informative.

**Suggested fix:** Remove the "Interpreting your question..." badge entirely. The command palette already shows a spinner (`<Loader2>`) when `isSearching` is true. That is sufficient. If the assistant query is actually executing, the results panel or a toast will surface the outcome.

---

### 2. "Searching live APIs..." — command-palette.tsx, line 328

```tsx
<span>Searching live APIs...</span>
```

The phrase "live APIs" is internal infrastructure jargon dressed up as a feature. Users do not think in terms of API calls; they think in terms of their tools. The badge also only appears when `liveSearchUsed` is true AND `isSearching` is true, meaning it disappears the moment results arrive, so it is only ever visible for a fraction of a second in practice.

**Suggested fix:** Remove the badge. The spinner already communicates "searching." If a distinction between cached and live results matters to the user, surface it on individual result rows (the "Live" chip on `PaletteSearchResultItem` already does this at the right granularity).

---

### 3. "Get a suggested answer" — command-palette.commands.tsx, line 340

```tsx
description: "Get a suggested answer",
```

"Suggested" here is filler. The description appears beneath "Ask assistant..." in the command list. "Suggested answer" implies the answer might be wrong or tentative, which is not the intended message. It also repeats the name without adding information.

**Suggested fix:** Remove the description entirely (it is nullable) or replace with something concrete: "Search your messages and tasks" or just omit it since the name "Ask assistant..." is self-explanatory.

---

### 4. "Ask assistant instead" empty-state CTA — command-palette.tsx, line 386

```tsx
Ask assistant instead
```

The word "instead" frames the assistant as a fallback after real search has failed, which undersells it. More importantly, the phrasing is passive. This is a minor wording issue rather than a heavy trope, but it compounds the pattern of positioning AI features as secondary.

**Suggested fix:** "Ask the assistant" without "instead", or more specifically: "Ask about this" to tie it to the query the user already typed.

---

### 5. "Cmd+K does everything" — command-palette-footer.tsx, line 28

```tsx
<span className="text-zinc-600">Cmd+K does everything</span>
```

This is marketing copy embedded in a UI hint footer. "Does everything" is a vague superlative that belongs in a landing page headline, not in a keyboard shortcut legend. It adds no information and reads as hype.

**Suggested fix:** Remove this string entirely. The footer already shows Navigate / Select / Esc hints, which are the only things a user needs while the palette is open.

---

## No findings

- command-palette-partial-failure-warning.tsx: clear, functional error messaging.
- command-palette-search-detail-panel.tsx: no trope language.
- command-palette.utils.tsx: utility functions only, no user-facing copy.
