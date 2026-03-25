# Trope Audit: apps/app/src/pages/oauth

Files audited:

- `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/oauth/page.tsx` (does not exist)

Note: The slice specified `oauth/page.tsx`. No file exists at that path. The actual OAuth callback handler lives at `oauth/callback/page.tsx`. That file was read and is audited below as it is the only user-facing content in this directory.

---

## oauth/callback/page.tsx

### User-facing strings inventory

| Location                            | Text                                                                      |
| ----------------------------------- | ------------------------------------------------------------------------- |
| Loading spinner label (auth)        | "Signing you in..."                                                       |
| Loading spinner label (integration) | "Securely connecting..."                                                  |
| Auth subtext                        | "Finishing authentication and loading your workspace."                    |
| Integration subtext                 | "Your credentials are encrypted and will only be used to sync your data." |
| Toast (auth success)                | "Signed in with Google"                                                   |
| Toast (integration success)         | "Connected to {displayName}"                                              |
| Toast (auth failure)                | "Sign in failed: {error}"                                                 |
| Toast (integration failure)         | "Failed to connect: {error}"                                              |
| Toast (missing handoff code)        | "Sign in failed: missing handoff code"                                    |
| Toast (handoff exchange failure)    | "Sign in failed: unable to exchange handoff"                              |
| Toast (token persistence failure)   | "Sign in failed: unable to store native session"                          |
| Toast (session restore failure)     | "Sign in failed: unable to restore session"                               |

### Analysis

Two strings are worth examining:

1. **"Securely connecting..."** -- The word "securely" is an adjective that adds reassurance rather than information. It does not tell the user what security means in this context. However, the subtext ("Your credentials are encrypted and will only be used to sync your data.") provides the concrete detail, so the pairing is acceptable.

2. **"Finishing authentication and loading your workspace."** -- "workspace" is standard SaaS vocabulary and not a trope here.

No AI writing tropes detected. No hype words, no em dashes, no vague capability claims.

**Findings:** None requiring change.
