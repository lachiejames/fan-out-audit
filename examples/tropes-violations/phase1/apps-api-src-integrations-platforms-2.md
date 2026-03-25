# Tropes Audit: apps/api/src/integrations/platforms (batch 2)

Files audited:

- `apps/api/src/integrations/platforms/google-docs/errors/google-docs.errors.ts`
- `apps/api/src/integrations/platforms/google-docs/utils/google-docs-error-mapper.ts`
- `apps/api/src/integrations/platforms/google-drive/errors/google-drive.errors.ts`
- `apps/api/src/integrations/platforms/google-drive/utils/google-drive-error-mapper.ts`
- `apps/api/src/integrations/platforms/google-gmail/errors/google-gmail.errors.ts`
- `apps/api/src/integrations/platforms/google-gmail/utils/google-gmail-error-mapper.ts`
- `apps/api/src/integrations/platforms/jira/errors/jira.errors.ts`
- `apps/api/src/integrations/platforms/jira/utils/jira-error-mapper.ts`

## Findings

### 1. "Please try again later" — google-docs.errors.ts, line 104

**Text**: `"Google Docs API rate limit exceeded. Please try again later."`

**Trope**: Vague, non-committal call to action. "Please try again later" gives the user no information about when or how to retry. It is a stock AI/support phrase.

**Fix**: State the constraint plainly. Example: `"Google Docs API rate limit hit. SlopWeaver will retry automatically."` or simply `"Google Docs API rate limit exceeded."` if no retry context is available.

---

### 2. "Please reconnect your Google account" — repeated pattern across google-docs.errors.ts

**Locations**:

- Line 75: `"Google Docs credentials are invalid. Please reconnect your Google account."`
- Line 87: `"Google Docs integration is not active (status: ${status}). Please reconnect your Google account."`
- Line 93: `"Google Docs integration not found. Please reconnect your Google account."`
- Line 109: `"Google Docs access token has expired. Please reconnect your Google account."`

**Trope**: Repetitive "Please reconnect" phrasing used as a catch-all. The word "Please" is deferential filler. Each message says the same thing with different preambles.

**Fix**: Drop "Please" and tighten. Examples:

- `"Google Docs credentials are invalid. Reconnect your Google account to continue."`
- `"Google Docs integration is inactive (status: ${status}). Reconnect your Google account."`
- `"Google Docs integration not found. Reconnect your Google account."`
- `"Google Docs access token expired. Reconnect your Google account."`

---

### 3. "Please reconnect your Google account" — repeated pattern across google-gmail.errors.ts

**Locations**:

- Line 156: `"Gmail credentials are invalid. Please reconnect your Gmail account."`
- Line 161: `"Gmail integration is not active (status: ${status}). Please reconnect your Gmail account."`
- Line 167: `"Gmail integration not found (required for People API access). Please reconnect your Gmail account."`

Same trope as above. "Please" adds no value and the phrasing is formulaic.

**Fix**: Same pattern as above — remove "Please", make the instruction direct.

---

### 4. "Please reconnect your account" — jira.errors.ts, line 210

**Text**: `"Failed to refresh Jira access token. Please reconnect your account."`

**Trope**: Same "Please reconnect" filler as above.

**Fix**: `"Jira access token refresh failed. Reconnect your account to continue."`

---

### 5. "Please free up space or upgrade your plan" — google-drive.errors.ts, line 138

**Text**: `"Google Drive storage quota exceeded. Please free up space or upgrade your plan."`

**Trope**: "Please" as deferential filler; "upgrade your plan" is a soft upsell phrasing commonly seen in AI-generated support copy.

**Fix**: `"Google Drive storage quota exceeded. Free up space or upgrade your Google storage plan."`

---

## No findings

The error-mapper files (`google-docs-error-mapper.ts`, `google-drive-error-mapper.ts`, `google-gmail-error-mapper.ts`, `jira-error-mapper.ts`) contain only code and JSDoc with no user-facing strings. No findings in those files.

The `jira.errors.ts` and `google-drive.errors.ts` factory messages (outside the ones flagged above) are terse and technical — no tropes found in those remaining messages.
