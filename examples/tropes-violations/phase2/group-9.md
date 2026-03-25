# Phase 2: Cross-Cutting Patterns (Group 9)

**Slices analyzed**: 12 (settings components 3-5, triage components 1-2, UI skeleton, lib contexts, inbox pages, AI pages 1-2, analytics, calls)

---

## Pattern 1: "Manage your X, Y, and Z" filler subtitles on settings sections

**Slices affected**: settings-3, pages-settings (2 of 12)

Multiple settings sections use the formula "Manage your [category list]" as a subtitle beneath the section heading. These subtitles add no information the heading does not already provide:

| File                           | Text                                           |
| ------------------------------ | ---------------------------------------------- |
| `settings-account.tsx` line 67 | "Manage your subscription, security, and data" |
| `settings-billing.tsx` line 16 | "Manage your subscription and usage"           |

**Systemic issue**: This is the standard AI-generated subtitle formula for settings pages. Every occurrence restates the heading in slightly different words. They can be removed or replaced with something concrete about what the user can actually do on that page.

---

## Pattern 2: "Please try again" as a generic error fallback

**Slices affected**: lib (ai-chat-controller, command-palette-context) (1 of 12, with 5+ occurrences)

The phrase "Please try again" appears as a fallback error description in at least five catch blocks across `ai-chat-controller.tsx` (lines 148, 554, 590) and `command-palette-context.tsx` (line 200). When no real error message is available, the UI shows this platitude instead of either omitting the description or displaying a more specific fallback.

| File                          | Occurrences             |
| ----------------------------- | ----------------------- |
| `ai-chat-controller.tsx`      | 3 (lines 148, 554, 590) |
| `command-palette-context.tsx` | 1 (line 200)            |

**Suggested fix**: When `error.message` is empty, omit the toast description entirely rather than printing a generic instruction. Users cannot meaningfully act on "Please try again" without knowing what went wrong.

---

## Pattern 3: AI-magic framing on empty states and cold-start screens

**Slices affected**: pages-ai-1 (CoreProfileSection, KnowledgeTab), pages-ai-2 (BehaviorTab), triage-1 (TriageStatsPanel), calls (calls-empty-state) (4 of 12)

Empty states across the app fill silence with vague promises about what the AI will eventually do, rather than telling the user what specific action triggers population:

| File                             | Text                                                                     | Issue                                             |
| -------------------------------- | ------------------------------------------------------------------------ | ------------------------------------------------- |
| `CoreProfileSection.tsx` line 39 | "Start using SlopWeaver and it will build your profile"                  | Vague, no concrete trigger                        |
| `KnowledgeTab.tsx` line 241      | "Your knowledge graph is growing"                                        | Implies activity when nothing may be happening    |
| `KnowledgeTab.tsx` line 243      | "accelerate graph growth"                                                | Marketing copy inside a product screen            |
| `BehaviorTab.tsx` line 83        | "Building your profile" / "enough messages are analyzed"                 | "Enough" is undefined, user cannot act on it      |
| `TriageStatsPanel.tsx` line 40   | "Complete your first session to see stats!"                              | Filler encouragement with exclamation mark        |
| `calls-empty-state.tsx` line 39  | "SlopWeaver uses call context to improve its understanding of your work" | AI justifying its own existence in an empty state |

**Systemic issue**: Empty states are the most common location for vague AI-magic copy. The pattern is: empty screen + promise about what the AI will learn + no specificity about what triggers it. The fix pattern is consistent: state what appears here and what action causes it to appear. Drop the AI-learning pitch.

---

## Pattern 4: "How your AI understands" and possessive-AI framing

**Slices affected**: pages-ai-1 (KnowledgeTab), pages-ai-2 (BehaviorTab, page.tsx) (2 of 12)

The AI profile pages repeatedly frame the AI as an autonomous entity that "understands" and "learns," positioning the user as an observer tracking the AI's growth:

| File                        | Text                                                                             |
| --------------------------- | -------------------------------------------------------------------------------- |
| `KnowledgeTab.tsx` line 324 | "Learning from your messages"                                                    |
| `BehaviorTab.tsx` line 57   | "See how your AI understands your work patterns"                                 |
| `BehaviorTab.tsx` line 421  | "How your AI understands your work patterns"                                     |
| `page.tsx` (AI) line 87     | "Your AI's knowledge base and behavioral observations. Track what it's learned." |

**Systemic issue**: The AI profile section is the densest cluster of anthropomorphic AI framing in the app. "Understands" implies comprehension rather than measurement. "Your AI" implies ownership of an agent. The AI pages should describe measured signals and extracted patterns, not frame the user as tracking a learning entity.

**Suggested fix**: Replace "understands" with "measures" or "tracks." Replace "Your AI's knowledge base" with "What SlopWeaver knows about you." Replace "Track what it's learned" with "Review extracted patterns."

---

## Pattern 5: "Detected" and robotic system language in notifications

**Slices affected**: triage-1 (TriageTriggerBanner) (1 of 12)

"Backlog detected" uses clinical system-fault language for what is a simple count threshold (25+ unread messages). The same banner uses a rhetorical question ("Triage them now?") as a soft CTA when the button already provides the action.

This is a narrow finding but representative of a broader tendency: using machine-report language ("detected," "classified") where plain language would be more natural.

---

## Pattern 6: Developer-internal language leaking to users

**Slices affected**: lib (command-palette-context) (1 of 12)

Three strings in `command-palette-context.tsx` expose developer jargon to end users:

| Text                          | Issue                                            |
| ----------------------------- | ------------------------------------------------ |
| "This action isn't wired yet" | Internal placeholder that may fire in production |
| "Running command..."          | "Command" is technical jargon                    |
| "Command complete"            | Same issue                                       |

**Suggested fix**: Either gate these behind dev-only conditions or replace with user-facing language ("Working...", "Done"). Consider suppressing the success toast entirely when no specific message is provided.

---

## Pattern 7: Loading state narration with trailing ellipsis

**Slices affected**: pages (inbox-messages-list, page.tsx) (1 of 12)

Loading labels like "Loading messages...", "Loading inbox...", and "Redirecting to setup..." use trailing ellipses that read as AI-narrated suspense. The pattern appears in at least three places in the inbox page group:

| File                              | Text                      |
| --------------------------------- | ------------------------- |
| `inbox-messages-list.tsx` line 48 | "Loading messages..."     |
| `page.tsx` (inbox) line 59        | "Loading inbox..."        |
| `page.tsx` (inbox) line 70        | "Redirecting to setup..." |

**Systemic issue**: Minor but consistent. Ellipsis on loading states is a common AI-era tic. Removing the trailing punctuation or switching to a progress indicator without text would be cleaner.

---

## Pattern 8: "In your style" and AI-safety reassurance as boilerplate

**Slices affected**: settings-3 (settings-integrations) (1 of 12)

`settings-integrations.tsx` line 157 combines two stock phrases: "reply in your style" and "nothing sends without you." Both are standard AI-product reassurance copy. "In your style" is a recurring trope across AI products, and "nothing sends without you" is the universal AI-safety disclaimer. These phrases appear in a loading state, which is the wrong moment for a value-prop pitch.

This is a single occurrence in this group, but based on earlier groups, "nothing sends without you" and "in your style" appear across multiple surfaces in the codebase.

---

## Pattern 9: Jargon in user-facing search results

**Slices affected**: settings-4 (settings-search) (1 of 12)

`settings-search.tsx` line 86 uses "deterministic" in a searchable item description visible to users. This is an engineering term that will mean nothing to most users encountering it in search results. The same word appears in `settings-rules.tsx` and `pages/settings/page.tsx`, but those contexts are deeper in settings where a technical audience is more likely. In search results, the audience is broader.
