# Trope Audit: apps/app/src/pages/forgot-password

Files audited:

- `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/forgot-password/loading.tsx`
- `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/forgot-password/page.tsx`

---

## loading.tsx

No user-facing text. Renders a skeleton component only.

**Findings:** None.

---

## page.tsx

### User-facing strings inventory

| Location                | Text                                             |
| ----------------------- | ------------------------------------------------ |
| Success state heading   | "Check your email"                               |
| Success state subtext   | "Reset link sent to {email}"                     |
| Resend button           | "Resend link" / "Sending..."                     |
| Back link               | "Back to sign in"                                |
| Form heading            | "Reset password"                                 |
| Form subtext            | "Enter your email and a reset link will be sent" |
| Submit button           | "Send reset link"                                |
| Resend prompt           | "Didn't receive it?"                             |
| Email placeholder       | "you@company.com"                                |
| Email label             | "Email"                                          |
| Error (server fallback) | "Failed to send reset link. Please try again."   |
| Error (resend fallback) | "Failed to resend reset link. Please try again." |
| Error (network)         | "Unable to connect to server. Please try again." |

### Analysis

All strings are functional and direct. No AI writing tropes detected:

- No "seamless", "powerful", "unlock", "supercharge", "game-changer", "revolutionary", or similar hype words.
- No em dashes.
- No vague AI promises.
- Error messages are plain and actionable.
- The success confirmation ("Reset link sent to {email}") is concrete, not effusive.

**Findings:** None.
