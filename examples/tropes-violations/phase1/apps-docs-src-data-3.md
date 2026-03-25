# Trope Audit: apps/docs/src/data/integrations (Slice 137)

**Files audited:**

- `apps/docs/src/data/integrations/shared.tsx`
- `apps/docs/src/data/integrations/slack.tsx`

---

## shared.tsx

Contains two privacy callout helper functions. User-visible text:

- `"{dataType} is encrypted in transit and at rest. Public models aren't trained on your data. See the Privacy Policy for details."`
- `"{text}"` (generic passthrough)

These are factual privacy disclosures. No marketing language. No AI tropes. No findings.

---

## slack.tsx

**intro:** "Connect your Slack workspaces to see DMs, mentions, and threads in the inbox. SlopWeaver syncs the messages you select and suggested replies you review."

Clean. Concrete. No violations.

**subtitle:** "Import your Slack workspaces into SlopWeaver"

Clean.

**connectStep 4 callout:** "SlopWeaver uses read-only access by default. SlopWeaver can only read messages, not send or delete them without your explicit action."

**MILD FLAG — Repetition of "SlopWeaver"**

Two consecutive sentences both start with "SlopWeaver." Minor prose issue, not a trope violation. The second sentence is factually clear.

**permissions entry:** "Allows the app to see which channels you're in and which threads you're part of."

Clean.

**syncExtra callout (Sync Focus):** "Adjust Sync Focus in Settings to widen or narrow what gets synced. Focused prioritizes your direct involvement; Broad widens to more channel context."

Clean documentation prose. No violations.

**syncCards:** Factual. No violations.

**No hard violations in slack.tsx.**

---

## Summary of Violations (Slice 137)

No significant trope violations in either file. `slack.tsx` has a minor prose repetition noted but no AI writing tropes. `shared.tsx` contains only factual privacy disclosure text.

---

## Cross-File Patterns Observed Across All Slices (133-137)

The following patterns repeat across multiple integration data files and are worth a single coordinated fix pass rather than file-by-file edits:

| Pattern                                                     | Files Affected                              | Recommended Action                                                         |
| ----------------------------------------------------------- | ------------------------------------------- | -------------------------------------------------------------------------- |
| `"without context-switching"` / `"without switching tools"` | jira.tsx, monday.tsx                        | Replace with concrete alternative in both                                  |
| `"across your workspace"`                                   | google-drive.tsx, microsoft-onedrive.tsx    | Remove padding phrase in both                                              |
| `"surface [X] context"`                                     | google-calendar.tsx, microsoft-calendar.tsx | Replace with "show" or rephrase to concrete action                         |
| `"in your voice"`                                           | linkedin.tsx                                | Replace — highest priority, this is a core brand trope that sounds generic |
| `"find what you need"`                                      | google-docs.tsx, notion.tsx                 | Rephrase to describe what is actually searched                             |
| `"stay on top of"`                                          | jira.tsx                                    | Replace with concrete phrasing                                             |
| `"alongside your other work"`                               | facebook-messenger.tsx                      | Remove — adds nothing                                                      |
| Mock search excerpts                                        | docs-search.tsx                             | Replace stock phrases with specific descriptions                           |
