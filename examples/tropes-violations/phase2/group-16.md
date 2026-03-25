# Phase 2: Cross-Cutting Patterns (Group 16)

**Slices analyzed**: 12 (packages/ui/src -- molecules slices 4-6, molecules/CitationChip, molecules/ConfidenceBar, molecules/ContextBar, molecules/EntityBadge, molecules/KnowledgeSparkles, molecules/PersonaSelector, molecules/SuggestionCard, molecules/stories, organisms/AiPresenceLogo)

---

## Pattern 1: Vague AI-framing nouns used as UI labels

**Slices affected**: KnowledgeSparkles, ContextBar, ConfidenceBar (3 of 12)

Multiple components use abstract AI-system vocabulary as user-visible labels instead of concrete descriptions of what is actually shown:

| Component              | String                                    | What it actually shows                                       |
| ---------------------- | ----------------------------------------- | ------------------------------------------------------------ |
| KnowledgeSparklesPanel | `"Insights"` (panel heading)              | Extracted facts about the user (email, tools used, patterns) |
| KnowledgeSparklesPanel | `"Insights appear as more data comes in"` | Empty state for fact extraction                              |
| ContextBar             | `"Context"` (section header)              | The specific email/task the AI is looking at                 |
| ConfidenceBar          | `"AI confidence: X%"` (aria-label)        | How likely the AI's suggestion matches the user's style      |

These labels treat "Insights" and "Context" as meaningful nouns, but they are AI-system-internal vocabulary that tells users nothing specific. The underlying data is concrete (facts, a specific email subject, a match score), but the labels abstract it away.

**Suggested fix**: Replace with concrete labels. "Insights" becomes "Facts" or "What we've learned". "Context" becomes the actual content title or "Viewing". The aria-label could specify what the confidence measures.

---

## Pattern 2: AI process language that frames routine work as discovery

**Slices affected**: KnowledgeSparkles, SuggestionCard (2 of 12)

Copy in these components frames ordinary data extraction and pattern matching as magical events:

| Component              | String                             | Issue                                                            |
| ---------------------- | ---------------------------------- | ---------------------------------------------------------------- |
| KnowledgeSparklesPanel | `"${totalCount} facts discovered"` | "Discovered" implies revelation; the system just parsed messages |
| KnowledgeSparklesPanel | `"Learning from your data..."`     | Vague process language with no actionable information            |
| SuggestionCard (JSDoc) | `"Urgent Mention Detected"`        | "Detected" frames the AI as a surveillance scanner               |

The verbs "discovered", "learning", and "detected" all inflate what the system is actually doing. The system found facts by parsing text, not by discovering hidden truths.

**Suggested fix**: Use plain verbs. "Found" instead of "discovered". "Processing messages" or "No facts yet" instead of "Learning from your data...". JSDoc examples should model action-oriented titles ("Reply needed from CEO") rather than detection-framing ("Urgent Mention Detected").

---

## Pattern 3: Anthropomorphized AI personas with human first names

**Slices affected**: PersonaSelector (1 of 12, but influences all persona UI downstream)

The PersonaSelector component's default data assigns human names to writing-style presets:

- `"Professional Sarah"` -- formal writing style
- `"Casual Alex"` -- casual writing style
- `"Technical Deep Dive"` -- technical analysis style
- `"Lightning Quick"` -- brief response style

The Storybook stories extend this with `"Marketing Maven"` and `"Legal Eagle"`. This anthropomorphization pattern creates fictional AI characters where functional labels ("Formal", "Casual", "Technical", "Brief") would be clearer and less cliche.

**Suggested fix**: Name presets by their communication style, not by fictional people. Drop human first names entirely.

---

## Pattern 4: Redundant "AI" qualifier in user-facing copy

**Slices affected**: PersonaSelector, ConfidenceBar (2 of 12)

The word "AI" appears in user-facing strings where it adds no information because the user already knows they are using an AI tool:

- `"Define your own AI communication style"` (PersonaSelector) -- "AI" is redundant
- `"AI confidence: X%"` (ConfidenceBar aria-label) -- the confidence bar is already in an AI context

**Suggested fix**: Remove "AI" where the context already makes it obvious. "Define your own communication style" and "Confidence: X%" are cleaner.

---

## Pattern 5: Structural molecules remain clean

**Slices affected**: molecules slices 4-6, CitationChip, EntityBadge, molecules/stories, AiPresenceLogo (7 of 12)

Seven of twelve slices had zero findings. The form-control family (6 sub-components), enhanced-card, flash, hover-card, inline-message, input-group, keybinding-hint, pagination, popover, progress-bar, select, sheet, tabs, toggle-group, chart, CitationChip, EntityBadge, and AiPresenceLogo all contain either no user-visible text or plain functional labels.

This confirms the pattern from Group 15: structural UI primitives are consistently clean. The trope violations cluster in AI-feature-specific molecules (KnowledgeSparkles, PersonaSelector, ConfidenceBar, ContextBar, SuggestionCard) rather than in generic UI building blocks.

---

## Summary

7 violations across 4 slices. All violations cluster in AI-feature-specific molecules. The dominant patterns are: (1) abstract AI-system vocabulary used as UI labels instead of concrete descriptions, (2) inflated verbs that frame routine processing as discovery, and (3) anthropomorphized persona names. Generic structural molecules remain clean.
