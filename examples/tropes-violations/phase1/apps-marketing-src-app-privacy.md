# Trope Audit: apps/marketing/src/app/privacy/page.tsx

Files audited:

- `apps/marketing/src/app/privacy/page.tsx`

---

## Findings

### Finding 1: "takes your privacy seriously"

- **Location**: line 119
- **Offending text**: `"SlopWeaver takes your privacy seriously."`
- **Why it's a trope**: This is one of the most overused opening sentences in privacy policy writing. Every company says it, including ones with poor privacy records. It signals nothing and damages credibility by association.
- **Suggested fix**: Drop the sentence entirely and lead directly with what the policy covers: `"This Privacy Policy explains how SlopWeaver collects, uses, stores, and protects your personal information."`

### Finding 2: Meta description — "Learn how your data is collected, used, and protected"

- **Location**: line 13–14
- **Offending text**: `"Privacy Policy for SlopWeaver. Learn how your data is collected, used, and protected when using the SlopWeaver platform."`
- **Why it's a trope**: "Learn how your data is collected, used, and protected" is the boilerplate meta description that appears on virtually every privacy policy page on the internet. It reads as template text.
- **Suggested fix**: Describe what makes SlopWeaver's data handling specific, e.g., `"SlopWeaver Privacy Policy. Covers integration data (Gmail, Slack, Linear), OAuth token storage, AI provider data handling, and your rights to export or delete."`

### Finding 3: "industry-standard practices"

- **Location**: line 141
- **Offending text**: `"Your data is stored securely using industry-standard practices:"`
- **Why it's a trope**: "Industry-standard practices" is a phrase that conveys nothing — it is the minimum bar, not a differentiator. The actual bullet points that follow (encrypted DB, TLS, encrypted OAuth tokens, RLS) are specific and good. The introductory phrase undercuts them.
- **Suggested fix**: `"Your data is stored with the following security controls:"` — then let the bullets speak for themselves.

### Finding 4: "Privacy-respecting analytics"

- **Location**: line 269
- **Offending text**: `"Privacy-respecting analytics are used with no cross-site tracking."`
- **Why it's a trope**: "Privacy-respecting" is a self-applied marketing label. Name the tool (PostHog, Plausible, etc.) and state what it does not collect. Self-declared privacy virtue is less credible than specific facts.
- **Suggested fix**: Name the analytics tool and its actual data practices, e.g., `"PostHog is used for product analytics. It does not track users across other sites and does not sell data."`

---

## Summary

| #   | File             | Line(s) | Trope                                                            | Severity |
| --- | ---------------- | ------- | ---------------------------------------------------------------- | -------- |
| 1   | privacy/page.tsx | 119     | "takes your privacy seriously" — the most cliched privacy opener | High     |
| 2   | privacy/page.tsx | 13–14   | Generic "learn how your data is collected" meta description      | Low      |
| 3   | privacy/page.tsx | 141     | "industry-standard practices" — vague minimum-bar claim          | Medium   |
| 4   | privacy/page.tsx | 269     | "Privacy-respecting analytics" — self-applied virtue label       | Medium   |
