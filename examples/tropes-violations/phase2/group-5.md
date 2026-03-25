# Phase 2: Cross-Cutting Patterns (Group 5)

**Slices analyzed**: 12 (integrations-platforms-2 through platforms-5, interface/agents, interface/attachment-embedding, interface/http, interface/push-notifications, shared/constants, shared/errors, shared/http, apps-app-src-components-1)

---

## Pattern 1: "Please reconnect your account" repeated mechanically across every platform

**Slices affected**: integrations-platforms-2, platforms-3, platforms-4, platforms-5 (4 of 12)

This is the continuation and confirmation of the dominant trope from Group 4. Every platform error file across all four batches uses `"Please reconnect your account."` or `"Please connect your [Platform] account."` as a formulaic suffix on credential, token, and integration errors. Affected platforms in this group: Google Docs, Google Gmail, Google Drive, Jira, Linear, LinkedIn, Microsoft Calendar, Microsoft OneDrive, Microsoft Teams, Notion, Slack.

Combined with Group 4's findings, this pattern spans all 15+ platform error files audited. It is the single largest source of AI-trope text in the codebase.

**Suggested fix**: Global search-and-replace: remove "Please" from all reconnect/connect messages in `apps/api/src/integrations/platforms/*/errors/*.errors.ts`. Change to direct imperatives: `"Reconnect your account."` or `"Reconnect your account to continue."`

---

## Pattern 2: "Please try again later" as stock rate-limit filler

**Slices affected**: integrations-platforms-2, platforms-3, platforms-4, platforms-5 (4 of 12)

Additional instances beyond Group 4: Google Docs, Microsoft Calendar, Microsoft Teams, Notion, Slack all use `"Please try again later."` in their rate-limit error messages. Combined with Group 4 (Facebook Messenger, GitHub, Google Calendar), this phrase appears in at least 8 platform error files.

**Suggested fix**: Remove `"Please try again later."` from all rate-limit messages. Replace with just the factual statement (`"[Platform] API rate limited."`) or surface the retry-after value when available.

---

## Pattern 3: Inconsistent trailing period and message framing across platforms

**Slices affected**: integrations-platforms-2, platforms-3, platforms-4 (3 of 12)

Cross-platform inconsistencies in the same error type:

| Error type     | Platform A                                                       | Platform B                                                                    |
| -------------- | ---------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| `unauthorized` | `"Please reconnect your Linear account"` (no period)             | `"Unauthorized access to LinkedIn API."` (trailing period, different framing) |
| `unauthorized` | `"Please reconnect your Microsoft Calendar account"` (no period) | LinkedIn uses a completely different sentence structure                       |
| `apiError`     | `"Linear API returned an error"` (no period)                     | `"LinkedIn API returned an error."` (period)                                  |
| `notFound`     | `"Linear comment with ID ... not found"` (no period)             | `"LinkedIn integration is not active."` (period)                              |

These messages surface to users in the same product UI but read as written by different authors, because each platform's error file was generated independently without a shared template.

**Suggested fix**: Create a shared error message template or constants for common error types (unauthorized, rate limited, not found, inactive). Each platform fills in the platform name; the sentence structure and punctuation are uniform.

---

## Pattern 4: "MicrosoftTeams" brand name typo in 7 user-facing strings

**Slices affected**: integrations-platforms-4 (1 of 12)

The Microsoft Teams error file uses `"MicrosoftTeams"` (no space) in 7 message strings: apiError, integrationInactive, integrationNotFound, oauthFailed, syncFailed, tokenExpired, unauthorized. This is a code-identifier-as-display-name leak, not technically a trope, but it signals auto-generated text that was never human-reviewed.

**Suggested fix**: Replace all `"MicrosoftTeams"` with `"Microsoft Teams"` in message strings in `microsoft-teams.errors.ts`.

---

## Pattern 5: "Please" politeness filler in shared error layer

**Slices affected**: shared/errors, integrations-platforms-\* (5 of 12)

The `shared/errors/message.errors.ts` file contains `"Please reconnect Google in Settings"` and `"Please connect your account in Settings"`, confirming that the "Please" filler pattern is not limited to platform error files but extends into the shared error layer. The `trialFeatureLocked` message also uses the stock upsell phrase `"Upgrade to unlock it."` -- "unlock" being a soft-sell cliche common in AI-generated product copy.

**Suggested fix**: Apply the same "Please" removal to shared error files. Change `"Upgrade to unlock it."` to `"This feature requires a paid plan."`

---

## Pattern 6: Hedged language ("may not have access")

**Slices affected**: integrations-platforms-4 (1 of 12, Notion)

Notion's `permissionDenied` error uses `"The integration may not have access to this resource."` The hedge "may not" is vague when the system knows the request was denied. This is a minor instance of the broader pattern where AI-generated text hedges instead of stating facts.

**Suggested fix**: `"The Notion integration does not have access to this resource."`

---

## Pattern 7: Frontend components contain AI chatbot cliches

**Slices affected**: apps-app-src-components-1 (1 of 12)

The AI chat components contain several stock chatbot UX patterns:

- `"What do you want to do?"` -- the generic "what can I help with" empty-state heading used by ChatGPT, Claude, and Gemini
- `"I can help with messages, tasks, replies, and more."` -- "and more" is filler that adds nothing; the list is generic enough to describe any AI assistant
- `"Approve this action?"` -- phrasing a prompt as a question instead of a statement

These are the first frontend findings across all five groups. They represent a different category of trope (chatbot UX cliche) from the backend pattern (over-polite error messages). The frontend copy should reflect SlopWeaver's specific positioning rather than defaulting to generic AI assistant language.

**Suggested fix**: Replace the empty-state heading and description with copy that reflects SlopWeaver's actual value proposition ("provable learning"). Replace `"Approve this action?"` with a statement like `"Action needs approval"`.

---

## Summary: Top 3 systemic patterns across Group 5

| Priority | Pattern                                       | Instances                                    | Scope                     |
| -------- | --------------------------------------------- | -------------------------------------------- | ------------------------- |
| 1        | "Please reconnect/connect your account"       | 30+ across all platform + shared error files | Global search-and-replace |
| 2        | "Please try again later" rate-limit filler    | 8+ platform error files                      | Global search-and-replace |
| 3        | Inconsistent message framing across platforms | All platform error files                     | Needs shared template     |
