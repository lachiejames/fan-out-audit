# Phase 2: Cross-Cutting Patterns (Group 10)

**Slices analyzed**: 12 (pages: contacts, forgot-password, integrations, login, oauth, onboarding, queue, reset-password, settings, signup, tasks, today)

---

## Pattern 1: "Unlock" framing and "unlimited" promises in paywall/billing copy

**Slices affected**: shared-context (PaywallContext, surfaced across multiple pages) (cross-cutting, sourced from 1 file but affects many pages)

The `PaywallContext.tsx` file (audited in the Group 11 shared-context slice but triggered from pages in this group) contains several paywall strings that use SaaS upsell tropes:

| Text                                          | Issue                                                                              |
| --------------------------------------------- | ---------------------------------------------------------------------------------- |
| "This feature is unlocked on paid plans"      | "Unlocked" frames the paid tier as removing a gate rather than providing a service |
| "Upgrade for unlimited usage" (2 occurrences) | "Unlimited" is likely inaccurate -- paid plans have limits too                     |
| "Upgrade anytime. Cancel anytime."            | Stock SaaS reassurance cliche used as a fallback subtitle                          |

These strings surface in upgrade modals triggered from pages across the app (tasks, today, AI, etc.), making them high-visibility despite living in a single context file.

**Suggested fix**: Replace "unlocked" with "available." Replace "unlimited usage" with "a higher limit" or the actual limit. Drop the "Cancel anytime" fallback or replace with a reason-specific message.

---

## Pattern 2: "Loading your day..." and anthropomorphic loading copy

**Slices affected**: today (page.tsx) (1 of 12)

"Loading your day..." treats a data fetch as something personal and experiential. This is a common AI product cliche -- framing a mundane operation as the AI doing something meaningful for the user. The trailing ellipsis adds to the effect.

This connects to the broader "loading state narration" pattern identified in Group 9 (inbox pages). The pattern is: loading states that narrate what the AI is doing rather than stating what is happening technically.

**Suggested fix**: "Loading..." or "Fetching today's brief..." -- describe the operation, not the experience.

---

## Pattern 3: Empty states promise emergent outcomes to motivate action

**Slices affected**: contacts (EntityGraphView), today (today-empty-state) (2 of 12)

Empty states across these pages fill silence with promises about future AI behavior:

| File                            | Text                                                    | Issue                                    |
| ------------------------------- | ------------------------------------------------------- | ---------------------------------------- |
| `EntityGraphView.tsx` line 325  | "Connect integrations to see your contact network grow" | Vague growth promise                     |
| `today-empty-state.tsx` line 99 | "based on your scheduled time"                          | References a schedule that may not exist |

This reinforces the Pattern 3 finding from Group 9: empty states are the most common location for vague AI-magic copy. The consistent fix is to state what appears and what action causes it.

---

## Pattern 4: "Smart Sort" and unexplained AI-flavored labels

**Slices affected**: tasks (page.tsx) (1 of 12)

The tasks page offers a "Smart Sort" option alongside concrete alternatives ("Newest First," "Priority"). "Smart" is a common AI product trope used when the actual sorting logic is unexplained. The label sits in a list of otherwise concrete options, making the vagueness conspicuous.

**Suggested fix**: Rename to describe the actual algorithm -- "Recommended," "By priority + date," or whatever the sort logic is.

---

## Pattern 5: Auth and form pages are consistently trope-free

**Slices affected**: forgot-password, login, oauth, reset-password, signup, queue (6 of 12)

Six of twelve slices had zero findings. The authentication flow (login, signup, forgot-password, reset-password), OAuth callback, and queue pages are consistently clean. Copy is functional, direct, and free of AI tropes. Error messages are specific and actionable. This is a positive pattern worth preserving.

The onboarding flow is also clean -- `welcome-step.tsx` is explicitly noted as a strong example of on-brand, trope-free copy with concrete, measurable claims ("your acceptance rate start to climb") and transparent constraints ("OAuth only, read-only by default").

---

## Pattern 6: "View #1" / "View #2" machine-generated link labels

**Slices affected**: today (morning-brief-panel.tsx) (1 of 12)

The morning brief panel renders related items with labels like "View #1" and "View #2" -- machine-generated labels that convey no meaning to the user. These should be replaced with the item title or a descriptive label. While not a writing trope per se, this is a UX copy failure common in AI-generated UI code.

---

## Pattern 7: Vague section headings that defer to content

**Slices affected**: today (morning-brief-panel, meeting-prep-section), tasks (tasks-calendar-panel) (2 of 12)

Several section headings use vague labels when more specific ones would serve the user better:

| File                                | Text                               | Better alternative                      |
| ----------------------------------- | ---------------------------------- | --------------------------------------- |
| `tasks-calendar-panel.tsx` line 177 | "Coming Up"                        | "Due Soon" or "Upcoming Due Tasks"      |
| `meeting-prep-section.tsx` line 128 | "Related"                          | "Related emails" or "Related documents" |
| `morning-brief-panel.tsx` line 274  | "Previous briefs will appear here" | "No previous briefs."                   |

These are minor individually but form a pattern of using soft, vague labels where concrete ones would be more informative.
