# Tropes Audit: apps/api/src/integrations/platforms (Linear, LinkedIn, Microsoft Calendar, Microsoft OneDrive)

Files audited:

- `apps/api/src/integrations/platforms/linear/errors/linear.errors.ts`
- `apps/api/src/integrations/platforms/linear/utils/linear-error-mapper.ts`
- `apps/api/src/integrations/platforms/linkedin/errors/linkedin.errors.ts`
- `apps/api/src/integrations/platforms/linkedin/utils/linkedin-error-mapper.ts`
- `apps/api/src/integrations/platforms/microsoft-calendar/errors/microsoft-calendar.errors.ts`
- `apps/api/src/integrations/platforms/microsoft-calendar/utils/microsoft-calendar-error-mapper.ts`
- `apps/api/src/integrations/platforms/microsoft-onedrive/errors/microsoft-onedrive.errors.ts`
- `apps/api/src/integrations/platforms/microsoft-onedrive/utils/microsoft-onedrive-error-mapper.ts`

---

## Findings

### 1. Trailing periods on some messages, absent on others (inconsistency)

Several messages end with a period while structurally identical sibling messages do not. This inconsistency signals auto-generated/AI-written text.

**With trailing period:**

- `linkedin.errors.ts:94` — `"LinkedIn API returned an error."`
- `linkedin.errors.ts:141` — `"Rate limited by LinkedIn API."`
- `linkedin.errors.ts:154` — `"Unauthorized access to LinkedIn API."`
- `linkedin.errors.ts:130` — `"Insufficient permissions for this LinkedIn action."`
- `linkedin.errors.ts:113` — `"LinkedIn integration is not active."`

**Without trailing period (same file, same pattern):**

- `linkedin.errors.ts:107` — `"LinkedIn credentials are invalid. Please reconnect your account."` (period mid-sentence, none at end — actually has one)
- `linear.errors.ts:85` — `"Linear comment with ID \"${commentId}\" not found"` (no period)
- `linear.errors.ts:103` — `"Linear issue with ID \"${issueId}\" not found"` (no period)
- `microsoft-calendar.errors.ts:145` — `"Calendar with ID \"${calendarId}\" not found"` (no period)
- `microsoft-calendar.errors.ts:157` — `"Calendar event with ID \"${eventId}\" not found"` (no period)

Recommendation: pick one convention and apply it uniformly. No trailing periods on error messages is the cleaner standard.

---

### 2. "Please" as a softening prefix on reconnect prompts

Multiple error messages use "Please" before an imperative action. This is a marker of AI-generated helpfulness copy and reads as overly deferential in an error context.

- `linear.errors.ts:97` — `"Linear integration not found. Please connect your Linear account."`
- `linear.errors.ts:108` — `"Linear access token expired. Please reconnect your Linear account."`
- `linear.errors.ts:111` — `"Please reconnect your Linear account"` (default for `unauthorized`)
- `linkedin.errors.ts:107` — `"LinkedIn credentials are invalid. Please reconnect your account."`
- `linkedin.errors.ts:118` — `"LinkedIn integration not found. Please reconnect your account."`
- `linkedin.errors.ts:148` — `"LinkedIn access token has expired. Please reconnect."`
- `microsoft-calendar.errors.ts:150` — `"Microsoft Calendar credentials are invalid. Please reconnect your account."`
- `microsoft-calendar.errors.ts:163` (integrationInactive) — `"...Please reconnect your account."`
- `microsoft-calendar.errors.ts:168` — `"Microsoft Calendar integration not found. Please reconnect your account."`
- `microsoft-calendar.errors.ts:184` — `"Microsoft Calendar access token has expired. Please reconnect your account."`
- `microsoft-calendar.errors.ts:188` — `"Please reconnect your Microsoft Calendar account"` (unauthorized)

Recommendation: drop "Please" and use direct imperatives. "Linear integration not found. Reconnect your Linear account." or just "Linear integration not found — reconnect to continue."

---

### 3. "Please try again later" filler

- `microsoft-calendar.errors.ts:179` — `"Microsoft Graph API rate limit exceeded. Please try again later."`

"Please try again later" is one of the most overused phrases in software error messages and is associated strongly with AI-generated filler. It provides no actionable information.

Recommendation: if retry timing is unknown, omit the suggestion. If a `retryAfter` value is available, surface it: "Rate limited. Retry after {N}s."

---

### 4. "Please free up space or upgrade your plan" — AI assistant suggestion padding

- `microsoft-onedrive.errors.ts:239` — `"OneDrive storage quota exceeded. Please free up space or upgrade your plan."`

The second sentence adds nothing specific and reads as generic AI helpfulness. The user already knows what "quota exceeded" means.

Recommendation: `"OneDrive storage quota exceeded."` — let the UI or documentation handle next steps.

---

### 5. Inconsistent message style across the same error type

`unauthorized` errors have different message shapes across platforms with no clear rationale:

- Linear: `"Please reconnect your Linear account"` (no period, no context sentence)
- LinkedIn: `"Unauthorized access to LinkedIn API."` (different framing, trailing period)
- Microsoft Calendar: `"Please reconnect your Microsoft Calendar account"` (no period)

These will surface to users in the same product context but read as written by different people (because they were generated separately). They should follow one template.

---

## Summary

| #   | File(s)                                     | Issue                                               | Severity |
| --- | ------------------------------------------- | --------------------------------------------------- | -------- |
| 1   | linkedin, linear, microsoft-calendar errors | Inconsistent trailing periods                       | Low      |
| 2   | All four platforms                          | "Please" prefix on reconnect messages               | Medium   |
| 3   | microsoft-calendar.errors.ts:179            | "Please try again later" filler                     | Medium   |
| 4   | microsoft-onedrive.errors.ts:239            | "Please free up space or upgrade your plan" padding | Low      |
| 5   | linkedin, linear, microsoft-calendar errors | Inconsistent `unauthorized` message framing         | Low      |

The error-mapper files (`*-error-mapper.ts`) contain no user-facing text — only HTTP status code mappings. No findings there.
