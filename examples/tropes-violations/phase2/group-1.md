# Phase 2: Cross-Cutting Patterns (Group 1)

**Slices analyzed**: 12 (`apps/api/src/application/` -- account-deletion, activity, analytics, auth, behavioral, billing, calendar, chat, clipboard-context, content, conversations, core-profile)

---

## Pattern 1: Internal jargon leaking into user-visible error messages

**Slices affected**: behavioral, chat, calendar, content, auth (5 of 12)

Error messages across the application layer expose internal implementation concepts that mean nothing to end users:

| Slice      | Example                                                 | Internal concept leaked            |
| ---------- | ------------------------------------------------------- | ---------------------------------- |
| behavioral | `Behavioral fingerprint not found for user "${userId}"` | "behavioral fingerprint", raw UUID |
| behavioral | `Insufficient data for behavioral analysis`             | "behavioral analysis"              |
| chat       | `Conversation "${conversationId}" not found`            | raw conversation ID                |
| calendar   | `Calendar config for user "${userId}" not found`        | raw user ID                        |
| auth       | `${resource} with ID "${id}" not found`                 | raw entity ID                      |
| content    | `Content with ID "${contentId}" not found`              | raw content ID                     |

**Systemic issue**: The `*.errors.ts` factory pattern across all modules creates error messages with template literals that embed raw UUIDs. These messages are returned in HTTP responses and can surface in the UI. Users seeing `Content with ID "a1b2c3d4-..." not found` get no useful information.

**Suggested fix**: Separate developer-facing detail (with IDs, for logs/debugging) from user-facing messages (plain language, no IDs). The error factories could carry both a `message` (user-safe) and a `detail` (developer-safe) field.

---

## Pattern 2: Developer instructions in user-reachable error messages

**Slices affected**: behavioral (1 of 12, but pattern likely exists in unaudited slices)

The behavioral slice contains `Use force=true to re-run`, which is an API parameter instruction embedded in an error message. This is a narrow instance of a broader risk: error messages written for the developer building against the API, not for the end user seeing the error in a UI.

Given that only the behavioral slice exhibited this in Group 1, this may be more widespread in integration or infrastructure layers (unaudited in this batch). Worth flagging as a pattern to watch for in later groups.

---

## Pattern 3: Overwhelming majority of backend error text is clean

**Slices affected**: 11 of 12 (all except behavioral)

Eleven of twelve slices had zero findings. The `application/` layer error messages are consistently terse, factual, and free of AI writing tropes (no hedging, no filler adverbs, no corporate softening, no apology language). This is a positive pattern worth preserving.

The behavioral module is the outlier, and its issues stem from domain-specific jargon rather than AI slop. The codebase does not have a systemic AI-writing-trope problem in backend error strings.

---

## Pattern 4: No user-facing prose exists in this layer

**Slices affected**: 12 of 12

None of these files contain marketing copy, onboarding text, notification bodies, email templates, or UI strings. All user-facing text is limited to short error messages returned in JSON API responses. The real trope risk is likely in:

- `apps/app/` (frontend UI copy)
- `apps/marketing/` (landing page)
- `apps/docs/` (user documentation)
- Email templates (if any exist in the API)
- AI-generated content (agent prompts, draft generation prompts)

This group of slices is low-risk for trope violations. Future audit effort should prioritize frontend and marketing layers.
