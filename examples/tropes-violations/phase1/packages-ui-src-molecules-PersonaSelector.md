# Tropes Audit: packages/ui/src/molecules/PersonaSelector

Files reviewed:

- `packages/ui/src/molecules/PersonaSelector/PersonaSelector.tsx`
- `packages/ui/src/molecules/PersonaSelector/PersonaSelector.stories.tsx`

---

## PersonaSelector.tsx

**Default persona data — names and descriptions (lines 71-108):**

| Name                    | Description                     |
| ----------------------- | ------------------------------- |
| `"Professional Sarah"`  | `"Formal, detailed responses"`  |
| `"Casual Alex"`         | `"Friendly, concise style"`     |
| `"Technical Deep Dive"` | `"Detailed technical analysis"` |
| `"Lightning Quick"`     | `"Ultra-brief responses"`       |

These are default persona definitions used when no external personas are provided. The names assign human-sounding first names to AI personas ("Professional Sarah", "Casual Alex"). This is a common AI product pattern of anthropomorphizing writing-style presets.

Verdict: **Violation.** Giving AI personas human first names is an AI-startup cliche that creates unnecessary fictional identities. The descriptions themselves are plain functional copy (no trope language), but the human-name framing ("Sarah", "Alex") implies a cast of AI characters rather than simply describing communication styles. More honest alternatives: "Formal" / "Casual" / "Technical" / "Brief" — or named by their function, not fictional people.

**"Create Custom Persona" button (line 266):**

> `"Create Custom Persona"`

Functional label. "Persona" is the product's chosen noun for writing-style presets. No findings.

**"Define your own AI communication style" (line 268):**

> `"Define your own AI communication style"`

The phrase "AI communication style" surfaces "AI" in user-facing copy. This is tolerable as a descriptor, but "your communication style" (without "AI") would be more direct and less jargony since the user is defining how the AI writes in their voice.

Verdict: Minor. The word "AI" here adds no information — the user already knows this is an AI tool. Remove "AI" for cleaner copy: `"Define your own communication style"`.

**"Manage Personas" link (line 281):**

> `"Manage Personas"`

Functional label. No findings.

**PersonaTraits JSDoc (lines 18-22):**

> `"How formal the responses are (0 = casual, 100 = professional)"` / `"How concise the responses are (0 = detailed, 100 = brief)"` / `"How technical the responses are (0 = simple, 100 = technical)"`

Internal developer documentation. No findings.

---

## PersonaSelector.stories.tsx

**Controlled story persona data (lines 38-55):**

> Names: `"Professional Sarah"` / `"Casual Alex"` — same human-name trope as default data above.
> Descriptions: `"Formal, detailed responses"` / `"Friendly, concise style"` — plain functional copy.

Same violation as in the component's default data. No new findings.

**CustomPersonas story (lines 74-92):**

> `"Marketing Maven"` / `"Growth-focused messaging"` / `"Legal Eagle"` / `"Precise, compliant language"`

Human-like persona names ("Marketing Maven", "Legal Eagle") continue the pattern of naming writing styles as fictional archetypes. Same violation category.

**FullFeatured story (lines 143-170):**

> `"Professional Sarah"` / `"Casual Alex"` / `"My Custom Style"` / `"Personalized communication"`

`"My Custom Style"` and `"Personalized communication"` are fine. The human names repeat the earlier violation. `"Personalized communication"` is vague but not a trope.

**Selected state display (lines 62, 186-189):**

> `Selected: <span>{selected.name}</span>` / `{selected.isCustom ? <span className="ml-2 text-amber-500">(Custom)</span> : null}`

Functional labels. No findings.

---

## Summary

**Violations found:**

1. **`PersonaSelector.tsx` lines 71-108 (default persona names)** — Using human first names ("Professional Sarah", "Casual Alex") for AI writing-style presets is an AI-startup anthropomorphization cliche. Writing styles should be named by their function ("Formal", "Casual", "Technical", "Quick") rather than as fictional people.

2. **`PersonaSelector.tsx` line 268** — `"Define your own AI communication style"` — "AI" is redundant and jargony here. Simplify to `"Define your own communication style"`.

3. **`PersonaSelector.stories.tsx` story data** — Story personas repeat the human-name trope ("Marketing Maven", "Legal Eagle") which will seed future copy. Stories should use functional names to avoid normalizing the pattern.
