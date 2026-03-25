# Trope Audit: apps/app/src/pages/login

Files audited:

- `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/login/loading.tsx`
- `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/login/page.tsx`

---

## loading.tsx

No user-facing text. Renders a skeleton component only.

**Findings:** None.

---

## page.tsx

### User-facing strings inventory

| Location                         | Text                                                                                        |
| -------------------------------- | ------------------------------------------------------------------------------------------- |
| Page heading                     | "Welcome back"                                                                              |
| Page subtext                     | "Sign in to your account."                                                                  |
| Email label                      | "Email"                                                                                     |
| Email placeholder                | "you@company.com"                                                                           |
| Password label                   | "Password"                                                                                  |
| Password placeholder             | "Enter your password"                                                                       |
| Forgot password link             | "Forgot password?"                                                                          |
| Submit button                    | "Sign in"                                                                                   |
| Signup prompt                    | "Don't have an account?"                                                                    |
| Signup link                      | "Create account"                                                                            |
| Error (native tokens missing)    | "Sign in failed: missing native session tokens. Please try again."                          |
| Error (native token persistence) | "Signed in, but failed to store your native session. Please restart the app and try again." |
| Error (credentials fallback)     | "Login failed. Please check your credentials."                                              |
| Error (network)                  | "Unable to connect to server. Please try again."                                            |
| Error (Google OAuth)             | "Unable to open Google sign in in your system browser. Please try again."                   |
| Turnstile error                  | "Please complete the verification challenge."                                               |

### Analysis

All strings are functional. One minor observation:

- **"Welcome back"** (h1): A common greeting convention for returning users. Not an AI trope per se, but it is a generic phrase that does not differentiate the product. Not a violation requiring action.
- No hype words, no em dashes, no vague AI promises.
- Error messages are plain, direct, and actionable.

**Findings:** None requiring change.
