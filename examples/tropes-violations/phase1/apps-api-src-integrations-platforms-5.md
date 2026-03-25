# Tropes Audit: Slice 51

## Files Audited

- `apps/api/src/integrations/platforms/slack/errors/slack.errors.ts`
- `apps/api/src/integrations/platforms/slack/utils/slack-error-mapper.ts`

## Findings

### `slack.errors.ts` — line 89

**String**: `"Slack API rate limited. Please try again later."`

**Trope**: "Please try again later" — filler phrase that adds no information. The user already knows they can try again later. This string is only reached when `retryAfterSeconds` is undefined, so a concrete retry time is not available, but the message still pads with pleasantry.

**Suggested fix**: `"Slack API rate limited."` or `"Slack rate limit hit."` — state the fact, omit the obvious instruction.

---

No other AI writing tropes found in these files. The fallback messages in `slack-error-mapper.ts` (`"Unhandled Slack error"`) are internal/developer-facing only and not user-visible.
