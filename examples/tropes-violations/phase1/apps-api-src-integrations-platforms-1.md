# Tropes Audit: apps/api/src/integrations/platforms (batch 1)

Files audited:

- `apps/api/src/integrations/platforms/asana/errors/asana.errors.ts`
- `apps/api/src/integrations/platforms/asana/utils/asana-error-mapper.ts`
- `apps/api/src/integrations/platforms/facebook-messenger/errors/facebook-messenger.errors.ts`
- `apps/api/src/integrations/platforms/facebook-messenger/utils/facebook-messenger-error-mapper.ts`
- `apps/api/src/integrations/platforms/github/errors/github.errors.ts`
- `apps/api/src/integrations/platforms/github/utils/github-error-mapper.ts`
- `apps/api/src/integrations/platforms/google-calendar/errors/google-calendar.errors.ts`
- `apps/api/src/integrations/platforms/google-calendar/utils/google-calendar-error-mapper.ts`

## Findings

### 1. "Please try again later" — vague filler phrase

**File**: `apps/api/src/integrations/platforms/facebook-messenger/errors/facebook-messenger.errors.ts`, line 175
**File**: `apps/api/src/integrations/platforms/github/errors/github.errors.ts`, line 113
**File**: `apps/api/src/integrations/platforms/google-calendar/errors/google-calendar.errors.ts`, line 115

These messages end with "Please try again later." without giving the user any actionable information about when or how to retry.

Facebook Messenger:

```
"Facebook API rate limited. Please try again later."
```

GitHub:

```
"Github API rate limited. Please try again later."
```

Google Calendar:

```
"Google Calendar API rate limit exceeded. Please try again later."
```

When `retryAfter` information is unavailable, the message should either be omitted or replaced with something concrete (e.g., "Rate limit hit. Retry in a few minutes." or just "Rate limited by [Platform] API."). "Please try again later" is a content-free filler that AI writing defaults to.

### 2. Redundant "Please" softening on reconnect prompts

**File**: `apps/api/src/integrations/platforms/asana/errors/asana.errors.ts`, lines 186, 249, 260
**File**: `apps/api/src/integrations/platforms/facebook-messenger/errors/facebook-messenger.errors.ts`, lines 157, 162, 169, 182
**File**: `apps/api/src/integrations/platforms/github/errors/github.errors.ts`, line 93
**File**: `apps/api/src/integrations/platforms/google-calendar/errors/google-calendar.errors.ts`, lines 86, 98, 104, 120

Error messages across all four platforms consistently append "Please reconnect your account." or "Please connect your account." This pattern is mechanically repeated and overly deferential for a system error. A direct instruction ("Reconnect your account.") or a factual statement ("Your session has expired.") is clearer and more confident. The word "Please" does not add meaning in an error message context.

Examples:

- `"Asana integration not found. Please reconnect your account."`
- `"Asana access token has expired. Please reconnect."`
- `"Unauthorized access to Asana. Please reconnect your account."`
- `"Facebook Messenger credentials are invalid. Please reconnect your account."`
- `"Facebook Messenger integration not found. Please connect your Facebook Page."`
- `"Facebook access token has expired. Please reconnect your Facebook Page."`
- `"Github integration not found. Please connect your Github account."`
- `"Google Calendar credentials are invalid. Please reconnect your account."`
- `"Google Calendar integration not found. Please connect your account."`
- `"Google Calendar access token has expired. Please reconnect your account."`

Note: GitHub's `unauthorized` factory (line 122) uses a default param of `"Please reconnect your Github account"` without the preceding context, making the message abrupt but still over-polite.

### 3. Inconsistent trailing period usage

**File**: `apps/api/src/integrations/platforms/asana/errors/asana.errors.ts`, line 180
**File**: `apps/api/src/integrations/platforms/facebook-messenger/errors/facebook-messenger.errors.ts`, line 163

Some messages end with a period, others do not, within the same file and across sibling platforms. For example, `"Asana integration is not active."` (has period) vs `"Asana API returned an error"` (no period). Facebook Messenger's `integrationInactive` ends with a period on the second sentence but not the first clause. This is minor inconsistency, not a trope per se, but reflects mechanical generation rather than deliberate editing.

## Summary

Three trope categories found across these eight files: vague "try again later" filler (3 instances across 3 platforms), over-polite "Please" on error messages that should be direct instructions (10+ instances across all 4 platforms), and minor punctuation inconsistency. No AI enthusiasm markers ("revolutionize", "seamless", "unlock", "powerful", "cutting-edge", "game-changer") or sycophantic openers found. No findings in the error-mapper files (they contain no user-facing strings).
