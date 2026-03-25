# Audit: apps-api-src-application-behavioral

**Files checked**

- `apps/api/src/application/behavioral/errors/behavioral.errors.ts`
- `apps/api/src/application/behavioral/utils/error-mapper.ts`

---

## Findings

### behavioral.errors.ts — line 69

**Category: ornate-nouns**

> `Insufficient data for behavioral analysis. Sample size: ${sampleSize}`

The phrase "behavioral analysis" is internal jargon that would reach users verbatim. A user seeing this error has no idea what "behavioral analysis" means in this product context. A plain alternative: "Not enough messages to analyze your writing style yet. Current count: ${sampleSize}."

---

### behavioral.errors.ts — line 77

**Category: serves-as-dodge**

> `Behavioral fingerprint not found for user "${userId}"`

Two problems. First, "behavioral fingerprint" is an internal implementation concept; users should never see that term. Second, embedding the raw UUID in a user-facing message is unhelpful and exposes internals. A plain alternative: "Writing style profile not found."

---

### behavioral.errors.ts — line 83

**Category: ornate-nouns**

> `Analysis was run ${hoursSinceLastAnalysis.toFixed(1)} hours ago. Use force=true to re-run.`

"Use force=true to re-run" is a developer-facing instruction leaking into what may be a user-visible error message. Users do not pass query parameters directly. This message assumes an API consumer rather than a product user. If this message surfaces in the UI, it needs a user-readable alternative. If it is developer-only, it is fine as-is but should be documented as such.

---

## No findings

- `error-mapper.ts` — contains only status-code mappings with no user-facing strings. Clean.
- No instances of magic-adverbs, delve-and-friends, negative-parallelism, stakes-inflation, em-dash-addiction, or bold-first-bullets in user-facing text.
