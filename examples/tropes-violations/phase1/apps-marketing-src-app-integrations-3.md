# Trope Audit: apps/marketing/src/app/integrations/ (Part 3)

**Slice:** 146
**Files audited:** 4

- `integrations/microsoft-teams/page.tsx`
- `integrations/monday/page.tsx`
- `integrations/notion/page.tsx`
- `integrations/slack/page.tsx`

All copy lives in `_config.tsx`. Individual `page.tsx` files are thin wrappers. Template-level violations documented in Part 1 (slice 144) and recurring pattern violations documented in Part 2 (slice 145) are referenced but not fully repeated here.

---

## Summary

Violations found, mostly recurring patterns from the rest of the integration suite. Slack and Teams are largely parallel (chat triage) and share the same structural weaknesses. Monday.com has an unusual phrasing error. Notion is clean.

---

## Violations

### 1. Microsoft Teams -- "Better Context" use case title (recurring pattern)

**File:** `_config.tsx`, microsoft-teams useCases
**Text:** "Better Context"

**Problem:** Same generic use case title documented in Parts 1 and 2. This is now the sixth-or-more instance of "Better Context" across integrations. At this point it is clearly a templated placeholder that was never individualized.

**Suggested fix (Teams-specific):** "Chat history next to inbox and tasks" or "Conversation context without the tab switch."

---

### 2. Monday.com -- useCasesDescription grammatical error

**File:** `_config.tsx`, monday useCasesDescription
**Text:** "Common ways Monday.com becomes easier to manage when board work stays inside SlopWeaver as the rest of your tools."

**Problem:** "as the rest of your tools" is grammatically broken. The intended meaning is "alongside the rest of your tools" or "as part of the rest of your tools." This is a copy error, not a trope, but it reads poorly and should be fixed.

**Suggested fix:** "Common ways Monday.com becomes easier to manage when board work stays alongside the rest of your tools in SlopWeaver."

---

### 3. Slack -- "Better Context" use case title (recurring pattern)

**File:** `_config.tsx`, slack useCases
**Text:** "Better Context"

**Problem:** Same generic use case title. Slack is the eighth integration to use "Better Context" as a use case section heading. At this scale it is a systemic copy problem rather than an individual one.

---

### 4. Teams and Slack parallel structure is not a violation but may confuse

**File:** `_config.tsx`, microsoft-teams and slack configs
**Observation:** Teams and Slack have near-identical use case titles ("Faster Chat Triage," "Better Context," "Cleaner Coordination," "Less Context Switching") and very similar descriptions. This is architecturally honest -- both are chat triage integrations -- but a visitor comparing the two pages would find almost no differentiation. The "aiLearns" field does differentiate them (Teams focuses on meeting follow-ups; Slack focuses on channel tone and DM-to-task conversion). Consider surfacing this difference more prominently.

This is an observation, not a trope violation.

---

### 5. Notion -- No significant violations

The Notion config uses the same structural template as other knowledge/docs integrations (Google Docs, Google Drive, OneDrive). The copy is accurate, restrained, and free of buzzwords. "Keep knowledge-base context close without hunting through another search surface" is specific. The "aiLearns" field ("Learns which pages you reference during specific workflows, your documentation patterns, and how Notion content informs decisions") is concrete.

No violations beyond the template-level issues already documented in Part 1.

---

## Cross-Integration Systemic Issues (Final Summary)

Across all three integration audit files (slices 144-146), the following recurring violations span the full integration suite and should be addressed systematically in `_config.tsx`:

| Pattern                                      | Count                         | Recommended action                           |
| -------------------------------------------- | ----------------------------- | -------------------------------------------- |
| "Better Context" as use case title           | 6+ integrations               | Replace with integration-specific titles     |
| "Never Miss Deadlines" as use case title     | 2 integrations (Asana, Gmail) | Replace with mechanism-based title           |
| "Unified Communication" as use case title    | 2 integrations (Asana, Gmail) | Replace with plain description               |
| "Stay Ahead of the Day" as use case title    | 2 calendar integrations       | Replace with factual statement               |
| "How It Works" section heading               | All 17 pages (template)       | Replace with "Technical details" or similar  |
| "Get Started Free" button label              | All 17 pages x2 (template)    | Evaluate whether contextual CTAs work better |
| "Full-Context Decisions" title               | GitHub                        | Replace with specific outcome                |
| Monday.com useCasesDescription grammar error | Monday.com                    | Fix "as the rest of your tools"              |
