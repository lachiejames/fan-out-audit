# Trope Audit: apps/marketing/src/app/security/page.tsx

Files audited:

- `apps/marketing/src/app/security/page.tsx`

---

## Overall Assessment

This security page is notably better than most. The "Honest Risk" section is candid and specific. The infrastructure details are factual. Findings below are minor.

---

## Findings

### Finding 1: "defense-in-depth principles"

- **Location**: line 123
- **Offending text**: `"The infrastructure is designed with defense-in-depth principles:"`
- **Why it's a trope**: "Defense-in-depth" is a real security term, but on a marketing page addressed to non-security professionals, it reads as jargon filler rather than explanation. The bullets that follow (encrypted DB, TLS, encrypted OAuth tokens, RLS) are the actual content; the phrase adds no information.
- **Suggested fix**: `"The infrastructure uses layered security controls:"` — or simply drop the introductory sentence and let the bullets stand alone.

### Finding 2: "You maintain control over data at all times"

- **Location**: line 147
- **Offending text**: `"You maintain control over data at all times:"`
- **Why it's a trope**: "You maintain control" is a reassurance phrase commonly used by data-heavy products to reduce anxiety. The bullets that follow (revoke access, account deletion, data export) are concrete and good. The intro phrase adds a marketing gloss that slightly undercuts the directness of the rest of the page.
- **Suggested fix**: `"Data controls available to you:"` — short and neutral, lets the bullets carry the message.

### Finding 3: Meta description — "privacy-first design"

- **Location**: line 14
- **Offending text**: `"Security practices at SlopWeaver. Learn how your data is protected with encryption, secure infrastructure, and privacy-first design."`
- **Why it's a trope**: "Privacy-first design" is a self-applied marketing label. The security page itself is more credible because it avoids such language in the body; the meta description should match that tone.
- **Suggested fix**: `"SlopWeaver security practices: encrypted database and OAuth tokens, TLS in transit, row-level security, and data controls. Honest about what's not yet audited."`

---

## No Finding: "Honest Risk" section

This section is explicitly NOT a trope violation. Calling out that this is a solo-developer operation without formal audits, and that granting access is a real tradeoff, is the opposite of AI writing tropes. It should be preserved and protected from future "improvement."

---

## Summary

| #   | File              | Line(s) | Trope                                                              | Severity |
| --- | ----------------- | ------- | ------------------------------------------------------------------ | -------- |
| 1   | security/page.tsx | 123     | "defense-in-depth principles" — jargon filler before good bullets  | Low      |
| 2   | security/page.tsx | 147     | "You maintain control over data at all times" — reassurance phrase | Low      |
| 3   | security/page.tsx | 14      | "privacy-first design" in meta description — self-applied label    | Low      |
