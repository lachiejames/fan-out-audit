# Trope Audit: apps/app/src/pages/signup

Files audited:

- `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/signup/loading.tsx`
- `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/signup/page.tsx`

---

## loading.tsx

No user-facing text. Renders a skeleton component only.

**Findings:** None.

---

## page.tsx

### User-facing strings inventory

| Location                         | Text                                                                              |
| -------------------------------- | --------------------------------------------------------------------------------- |
| Page heading                     | "Create your account"                                                             |
| Page subtext                     | "Connect your first inbox next."                                                  |
| Email label                      | "Email"                                                                           |
| Email placeholder                | "you@company.com"                                                                 |
| Password label                   | "Password"                                                                        |
| Password hint                    | "Must be at least 8 characters"                                                   |
| Password placeholder             | "Create a strong password"                                                        |
| Terms checkbox label             | "I agree to the"                                                                  |
| Terms link                       | "Terms of Service"                                                                |
| Policy link                      | "Privacy Policy"                                                                  |
| Submit button                    | "Create account"                                                                  |
| Sign in prompt                   | "Already have an account?"                                                        |
| Sign in link                     | "Sign in"                                                                         |
| Error (turnstile)                | "Please complete the verification challenge."                                     |
| Error (terms)                    | "Please agree to the Terms of Service and Privacy Policy"                         |
| Error (email validation)         | "Please enter a valid email"                                                      |
| Error (password too short)       | "Password needs at least 8 characters"                                            |
| Error (native tokens missing)    | "Account created but native session tokens are missing. Please sign in again."    |
| Error (native token persistence) | "Account created, but failed to store your native session. Please sign in again." |
| Error (server fallback)          | "Signup failed. Please try again."                                                |
| Error (network)                  | "Unable to connect to server. Please try again."                                  |
| Error (Google OAuth)             | "Unable to open Google sign up in your system browser. Please try again."         |
| Error (browser open)             | "Unable to open that page in your system browser. Please try again."              |

### Analysis

- **"Create your account"** (h1): Direct, functional. No trope.
- **"Connect your first inbox next."**: This subtext is functional -- it sets an expectation for the next step (onboarding). "inbox" is product-specific vocabulary, not a generic AI hype word. No trope.
- **"Create a strong password"** (placeholder): Common guidance pattern. No trope.
- All error messages are specific and actionable.
- No hype words, no em dashes, no vague AI capability claims.

**Findings:** None.
