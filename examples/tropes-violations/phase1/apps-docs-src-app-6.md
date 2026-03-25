# Tropes Audit: apps/docs/src/app — Slice 6

**File audited**:

- `src/app/integrations/linkedin/page.tsx`

---

## Structure note

This file follows the same `IntegrationPageTemplate` wrapper pattern as Slices 4 and 5. Only the SEO metadata `description` field contains user-visible text in this file.

---

## Metadata description audited

**LinkedIn**:

> "Connect LinkedIn to create posts directly from SlopWeaver. Your AI assistant drafts LinkedIn content in your voice for approval before publishing."

---

## Findings

**"in your voice"**: This is a mild marketing phrase common in AI writing-tool copy. "Drafts LinkedIn content that matches your style" would be more concrete and testable. "In your voice" is vague and has become a cliche in the AI writing space.

**Severity**: Minimal.

---

## Summary for Slice 6

| File              | Findings                                                          |
| ----------------- | ----------------------------------------------------------------- |
| linkedin/page.tsx | 1 minimal finding: "in your voice" cliche in metadata description |

---

# Cross-Cutting Patterns — Full Docs Site

Having audited all 41 `.tsx` files under `src/app/`, the following cross-cutting patterns are worth flagging at the site level:

## 1. Repetitive fractal summaries across pages

Multiple pages end with a callout that restates a general principle already covered by the page body, often with an Analytics link. Examples:

- analytics/page.tsx: "Even early numbers show the AI is working and learning."
- behavioral/page.tsx: "The behavioral fingerprint is the foundation of all AI personalization."
- contacts/page.tsx: "Entity resolution powers the AI's per-recipient style matching... This contributes to higher acceptance rates."
- inbox/page.tsx: "Every message you read...feeds the AI's understanding."
- knowledge-sources/page.tsx: Knowledge sources accelerate learning — restated from the intro.
- morning-brief/page.tsx: "The more platforms you connect...the more relevant your briefs become."
- meeting-prep/page.tsx: "Meeting prep improves as the AI learns more about your work."
- triage/page.tsx: "Override learning is one of the most direct ways to train the AI."
- queue/page.tsx: "Every approval, edit, or rejection...trains the AI."
- search/page.tsx: "Search results improve as the AI processes more of your data."
- tasks/page.tsx: "Completing AI-extracted tasks contributes to your time-saved metric."

**Pattern**: Nearly every feature page ends with a callout saying (a) the AI learns from your interactions, and (b) check Analytics. This creates a fractal-summary effect at scale — the closing callout is predictable and adds no page-specific information.

**Recommendation**: Reserve the Analytics-link callout for pages where it is genuinely page-specific (e.g., analytics/page.tsx itself, queue/page.tsx where acceptance rate is central). Remove or replace it on pages where it is a generic append.

## 2. "One shortcut to rule them all" — duplicate across two pages

This phrase appears identically in both `keyboard-shortcuts/page.tsx` and `search/page.tsx`. One should be removed or reworded.

## 3. Integration page metadata is clean overall

The 17 integration page metadata descriptions are factual and specific. Minor issues (noted in Slices 4-6) are isolated. No systematic tropes problem in integration metadata.

## 4. No "delve", "leverage", "robust", "seamlessly", "transformative" found

A keyword scan of all audited files found none of the classic AI writing magic adverbs or luxury verbs in user-facing prose.

## 5. No em-dashes in user-facing body copy

Em-dashes do not appear in body text. Some UI strings use hyphens appropriately.

## 6. No "imagine a world" or false-suspense openers

No pages use scene-setting openers or false-suspense hooks.
