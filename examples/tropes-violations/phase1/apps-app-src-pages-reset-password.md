# Trope Audit: apps/app/src/pages/reset-password

Files audited:

- `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/reset-password/loading.tsx`
- `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/reset-password/page.tsx`

---

## loading.tsx

No user-facing text. Renders a skeleton component only.

**Findings:** None.

---

## page.tsx

### User-facing strings inventory

| Location                        | Text                                                                                  |
| ------------------------------- | ------------------------------------------------------------------------------------- |
| Success state heading           | "Password updated"                                                                    |
| Success state subtext           | "Your password has been successfully reset"                                           |
| Success CTA                     | "Sign in with new password"                                                           |
| Form heading                    | "Set new password"                                                                    |
| Form subtext                    | "Choose a strong password for your account"                                           |
| New password label              | "New password"                                                                        |
| New password placeholder        | "Create a strong password"                                                            |
| Confirm password label          | "Confirm password"                                                                    |
| Confirm password placeholder    | "Re-enter your password"                                                              |
| Requirements label              | "Password requirements:"                                                              |
| Requirement items               | "At least 8 characters", "One uppercase letter", "One lowercase letter", "One number" |
| Submit button                   | "Update password"                                                                     |
| Error (invalid/expired link)    | "Reset link is invalid or expired. Please request a new one."                         |
| Error (server fallback)         | "Failed to reset password. Please try again."                                         |
| Error (network)                 | "Unable to connect to server. Please try again."                                      |
| Validation (passwords mismatch) | "Passwords do not match"                                                              |
| Validation (confirm required)   | "Please confirm your password"                                                        |
| Validation (too short)          | "Password needs at least 8 characters"                                                |

### Analysis

All strings are functional and direct. No AI writing tropes detected:

- No hype words, no em dashes, no vague capability claims.
- "Create a strong password" as a placeholder is a common convention giving useful guidance, not marketing language.
- "Choose a strong password for your account" as subtext is informational only.
- Error messages are plain, specific, and actionable.

**Findings:** None.
