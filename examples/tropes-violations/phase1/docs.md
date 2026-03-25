# Trope Audit: docs/CONTENT-PLAYBOOK.md and docs/POSITIONING.md (Slice 160)

Files audited:

- docs/CONTENT-PLAYBOOK.md
- docs/POSITIONING.md

These are internal guidance documents, not public-facing copy. The audit evaluates whether the documents themselves contain the tropes they are meant to prohibit, and whether the guidance they encode is internally consistent and complete.

---

## CONTENT-PLAYBOOK.md

### Document purpose

Tactical copy guide for all contexts. Defines banned phrases, tone by channel, and the provable learning test.

### Findings

**Lines 57-65 (Banned phrases list) vs line 104 in the pill badge audit (pricing-cards-section.tsx)**
The playbook bans "AI that learns you" as a category label? No, it bans the old console tagline and inbox-aggregator framing. The pill badge "AI that learns you" (found in pricing-cards-section.tsx) is a shortened form of the canonical tagline "The AI that proves it's learning you" and is not explicitly banned in the playbook. However, the shortened form drops the critical word "proves," which is the whole differentiator. The playbook does not address shortened tagline forms. This is a gap: the guidance should explicitly state that shortening the tagline to "AI that learns you" loses the proof claim and is not an acceptable shorthand.

**Lines 79-86 (Structural AI tells)**
The playbook bans "Negative parallelism: 'It's not X. It's Y.'" and "Dramatic countdown: 'Not X. Not Y. Just Z.'"
These are well-chosen bans. No violations found in the playbook itself.

**Lines 110 (Formatting bans)**
"Em dashes: BANNED. Use commas, periods, or parentheses instead."
The playbook itself contains no em dashes. Consistent.

**Lines 97-106 (Outdated positioning / retired descriptors)**
The playbook refers to "The old console tagline" without naming it. This is intentionally vague ("acceptable only in technical CLAUDE.md sections"). The vagueness is fine for the purpose, but a new contributor would not know what "the old console tagline" is without context. Minor documentation gap, not a trope violation.

**Lines 278-285 (Numbers and Specifics table)**
The table maps vague claims to specific preferred alternatives. Row: `"Affordable pricing" | "Starts at $39/month with 3,000 actions"`.
This is good guidance, but the $39 figure and 3,000 actions count should be verified against the actual contract constants (`SUBSCRIPTION_TIER_DISPLAY.starter.monthlyPrice`, `SUBSCRIPTION_TIER_DISPLAY.starter.monthlyActions`). If those values change, this hardcoded example becomes stale. Not a trope violation but a maintenance risk.

**Lines 291-297 (Humor Policy)**
`"OK: Self-deprecating honesty ('We're a solo-dev operation. The AI is smarter than our marketing.')"`
This example is fine as internal guidance. However, "The AI is smarter than our marketing" is itself a mild claim about the AI's capabilities. Acceptable in context (it is a humility statement, not an overclaim).

**No structural AI tells found in the document itself.** The playbook does not open with "In today's fast-paced world," does not use dramatic countdowns, and avoids the anaphora abuse it bans.

---

## POSITIONING.md

### Document purpose

Canonical positioning reference. Defines product category, 5-pillar messaging hierarchy, competitive positioning, and brand voice.

### Findings

**Line 11** - `"Not 'unified inbox.' Not 'AI assistant.' Not 'productivity tool.'"`
This is a negative parallelism structure ("Not X. Not Y. Not Z."), which the CONTENT-PLAYBOOK.md bans under "Dramatic countdown: 'Not X. Not Y. Just Z.'" The POSITIONING.md uses this structure in internal guidance, which is probably intentional (the point is to say what SlopWeaver is NOT). However, the distinction between "banned in copy" and "acceptable in internal docs" is not stated. If an agent quotes this line into marketing copy it would violate the playbook. Worth adding a note that this negative parallelism is for internal reference only and must not appear in marketing output.

**Lines 97-126 (Competitive Positioning)**
All five competitive comparisons use the format: `"[Competitor] does X. SlopWeaver does Y."` This is a contrast structure, which is different from the banned "It's not X. It's Y." reframe, but agents should be careful not to lift these lines directly into marketing copy where competitors should not be named. The positioning doc correctly marks these as internal-only by implication, but could state it explicitly.

**Line 99** - `"Copilot's accuracy NPS is -19.8."`
This is a specific competitive claim with no source cited. If this is a real figure it needs a citation so it can be verified and updated. If it is illustrative or from a specific survey, the context should be noted. Publishing this uncited in any external channel would be a risk.

**Lines 130-140 (Top 10 Differentiators)**
Uses bold-first bullet structure: `**Provable learning metrics**: ...`, `**Cross-platform behavioral fingerprint**: ...` etc. This is the "Bold-first bullet lists where every item starts with a bolded keyword" pattern that the CONTENT-PLAYBOOK.md bans under "Structural AI tells." The ban in the playbook applies to "everywhere," but these are internal docs. Same issue as the negative parallelism above: the ban should distinguish internal reference docs from external copy, or the internal docs should use a different structure to avoid normalizing the pattern.

**Lines 170-178 (Brand Voice Summary)**
The summary itself is clean, direct, and consistent with the tone it describes. No violations.

**No banned phrases detected in the document itself.** No "leverage," "streamline," "seamless," "revolutionary," "game-changing." The document practices what it preaches.

**No em dashes detected.** Consistent with the formatting ban.

---

## Cross-document consistency check

| Claim in POSITIONING.md                                    | Consistent with CONTENT-PLAYBOOK.md?                                                    |
| ---------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| "The AI that proves it's learning you" as primary category | Yes                                                                                     |
| "Chief of Staff" as supporting analogy only                | Yes                                                                                     |
| Anti-hype is a brand value                                 | Yes                                                                                     |
| No em dashes                                               | Yes, both documents comply                                                              |
| Specific numbers over vague claims                         | Yes, both enforce this                                                                  |
| Competitor naming rules                                    | Playbook says "never in marketing copy"; positioning uses names internally. Consistent. |

---

## Summary

| Document            | Hard violations | Gaps / Risks                                                                                                                                                                                                |
| ------------------- | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CONTENT-PLAYBOOK.md | 0               | Hardcoded $39/3,000 actions example may go stale; shortened tagline "AI that learns you" not explicitly addressed                                                                                           |
| POSITIONING.md      | 0               | Negative parallelism structure on line 11 could be mistakenly copied to marketing; competitive NPS claim on line 99 has no citation; bold-first bullet structure in Top 10 list normalizes a banned pattern |

**Priority fixes:**

1. `POSITIONING.md` line 99: Add source or date for Copilot NPS figure, or remove if unverifiable.
2. `POSITIONING.md` lines 11 and 130: Add an inline note that the negative parallelism and bold-first bullet structures are internal reference formatting and must not appear in external copy.
3. `CONTENT-PLAYBOOK.md`: Add explicit guidance that "AI that learns you" (the shortened tagline form without "proves") is not an acceptable shorthand in any channel.
4. `CONTENT-PLAYBOOK.md` Numbers table: Source the $39/3,000 actions example from contract constants rather than hardcoding it.
