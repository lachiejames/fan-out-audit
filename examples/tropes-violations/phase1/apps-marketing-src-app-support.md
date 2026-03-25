# Trope Audit: apps/marketing/src/app/support/

Files audited:

- `apps/marketing/src/app/support/page.tsx`
- `apps/marketing/src/app/support/support-form.tsx`

---

## support/page.tsx

### Finding 1: "Get in touch" heading

- **Location**: line 75
- **Offending text**: `<h2>Get in touch</h2>`
- **Why it's a trope**: "Get in touch" is one of the most overused contact section headings in SaaS. It is generic to the point of invisibility.
- **Suggested fix**: Something specific to the solo context, e.g., `"Send a message"` or, leaning into the page's own framing, `"Reach the person who built it"`.

### Finding 2: "Send a message and we'll get back to you within 1 business day"

- **Location**: line 77
- **Offending text**: `"Send a message and we&#39;ll get back to you within 1 business day."`
- **Why it's a trope**: The "we'll" here implies a team when the rest of the page correctly establishes this is a solo product. The page header says "Real help from the person who built this" but the form subtitle reverts to corporate "we" language.
- **Suggested fix**: `"Send a message. You'll hear back within 1 business day — from me."` Matches the solo-founder framing consistently.

### Finding 3: FAQ — "Is my data secure?" answer — "industry-standard" echoed implicitly

- **Location**: lines 51–53
- **Offending text**: `"All data is encrypted in transit (TLS 1.2+) and at rest. OAuth tokens are encrypted with AES-256-GCM. Row-Level Security enforces tenant isolation at the database level. We do not sell or share your data, and we do not use it to train AI models. See our security page for full details."`
- **Why it's a note (not a hard flag)**: This FAQ answer is actually good — specific algorithm names, no "industry-standard" filler. The "we do not" constructions are appropriate for FAQ context. No change required.

### Finding 4: "How does the AI work?" answer — "Every AI-suggested action requires your explicit approval"

- **Location**: lines 55–58
- **Offending text**: `"AI features like draft replies, triage, and task extraction use Anthropic Claude. Every AI-suggested action requires your explicit approval before anything is sent, modified, or deleted. Nothing happens automatically."`
- **Why it's borderline**: "Nothing happens automatically" is a fine reassurance that is factually accurate and specific to SlopWeaver's model. Not a trope per se. However "requires your explicit approval" paired with "before anything is sent, modified, or deleted" repeats the same point twice. The sentence is longer than it needs to be.
- **Suggested fix**: `"Draft replies, triage, and task extraction use Anthropic Claude. Every suggested action sits in an approval queue — nothing sends without you."`

### Finding 5: Philosophy section — closing quote

- **Location**: lines 164–167
- **Offending text**: `"My goal is to build a product that reduces the need for you to reach out in the first place. But when you do need help, you'll get a real answer from the person who built it."`
- **Why it's a note**: This is genuine and on-brand. The first sentence is particularly good. No change required. Flagging only to note it cleared review.

---

## support-form.tsx

No user-visible marketing copy. The form contains only functional UI text: field labels ("Name", "Email", "Category", "Message"), placeholder text ("Your name", "you@company.com", "Describe your issue or question..."), button states ("Sending...", "Send message"), and status messages ("Message sent. We'll get back to you within 1 business day.").

### Finding 6: Success message — "we'll" again

- **Location**: line 152
- **Offending text**: `"Message sent. We&#39;ll get back to you within 1 business day."`
- **Why it's a trope**: Same as Finding 2 above — "we'll" implies a team on a solo product. Should match the page-level messaging.
- **Suggested fix**: `"Message sent. You'll hear back within 1 business day."` or `"Message sent. I'll get back to you within 1 business day."`

---

## Summary

| #   | File             | Line(s) | Trope                                                                       | Severity |
| --- | ---------------- | ------- | --------------------------------------------------------------------------- | -------- |
| 1   | support/page.tsx | 75      | "Get in touch" — generic contact heading                                    | Low      |
| 2   | support/page.tsx | 77      | "we'll get back to you" — corporate plural on a solo product                | Medium   |
| 3   | support/page.tsx | 55–58   | Doubled reassurance ("explicit approval" + "nothing happens automatically") | Low      |
| 4   | support-form.tsx | 152     | "We'll get back to you" repeated in success message                         | Medium   |
