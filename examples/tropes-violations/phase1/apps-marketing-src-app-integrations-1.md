# Trope Audit: apps/marketing/src/app/integrations/ (Part 1)

**Slice:** 144
**Files audited:** 8

- `integrations/integration-marketing-page.tsx` (does not exist at specified path; actual location is `integrations/_components/integration-marketing-page.tsx`)
- `integrations/_config.tsx` (the source of all integration copy -- audited in full for this and subsequent slices)
- `integrations/page.tsx`
- `integrations/asana/page.tsx`
- `integrations/facebook-messenger/page.tsx`
- `integrations/github/page.tsx`
- `integrations/google-calendar/page.tsx`
- `integrations/google-docs/page.tsx`

Note: The individual `page.tsx` files for each integration are thin wrappers that render `IntegrationMarketingPage` with config from `_config.tsx`. All copy lives in `_config.tsx`. The `_components/integration-marketing-page.tsx` contains the shared rendering template. Violations are reported against the config entries and template labels as shown to users.

---

## Summary

Moderate violations found. The integration pages share a structural template that has some problematic recurring patterns. Most individual integration copy is factual and grounded. The main issues are template-level phrases that repeat across all 17 integrations and a few per-integration copy choices.

---

## Violations

### 1. "How It Works" section heading (template-level, affects all 17 pages)

**File:** `_components/integration-marketing-page.tsx`, line 79
**Text:** "How It Works"

**Problem:** "How It Works" is the most generic section heading in SaaS. It says nothing about what specifically will be explained. Every integration uses this same heading for the OAuth/API technical details section.

**Suggested fix:** More specific heading tied to the content, e.g., "The technical setup" or simply "Technical details" or "How the connection works."

---

### 2. "Get Started Free" button label (template-level, appears twice per page)

**File:** `_components/integration-marketing-page.tsx`, lines 43, 167
**Text:** "Get Started Free"

**Problem:** Stock SaaS CTA label. Appears on every integration page in both the hero and the footer CTA. Not catastrophic but contributes to template blandness.

**Suggested fix:** Consider contextual CTAs like "Connect [Platform]" or "Try it free."

---

### 3. Integrations index page description oversells

**File:** `integrations/page.tsx` (metadata)
**Text:** "Every tool you connect gives your AI more to learn from. More data means faster calibration, higher accuracy, better predictions."

**Problem:** "Better predictions" is vague. "Faster calibration, higher accuracy, better predictions" reads as a list of abstract AI marketing claims without specificity. What predictions? Accurate at what?

**Suggested fix:** Ground it: "Connect Gmail and Slack and your AI learns whether you're formal with your manager in email but casual in chat. More tools, more signal, higher acceptance rate."

---

### 4. Integrations index page body text

**File:** `integrations/page.tsx`, line 66
**Text:** "Every tool you connect gives your AI more to learn from. More data means faster calibration, higher accuracy, better predictions. All 17 integrations are available now."

**Problem:** Same as metadata issue above. "Better predictions" is generic. The metadata and the on-page text are near-identical, which wastes an opportunity to say something different in each context.

---

### 5. Asana -- "Never Miss Deadlines" use case title

**File:** `_config.tsx`, Asana useCases
**Text:** "Never Miss Deadlines"

**Problem:** Stock productivity-software promise. Every task app claims this. It is not differentiated and implies a reliability guarantee SlopWeaver cannot make.

**Suggested fix:** Describe the actual mechanism: "Tasks with due dates stay surfaced" or "Deadlines and follow-ups become trackable items."

---

### 6. Asana -- "Unified Communication" use case title

**File:** `_config.tsx`, Asana useCases
**Text:** "Unified Communication"

**Problem:** "Unified Communication" is UCaaS industry jargon. Asana is a task tool, not a communication platform. The title does not match the description (which is about handling Asana alongside other tools in one view). This appears to be a copy-pasted title from an email/chat integration context.

**Suggested fix:** "One operating view" or "Asana alongside the rest of your stack."

---

### 7. Facebook Messenger -- heroDescription passive construction

**File:** `_config.tsx`, facebook-messenger heroDescription
**Text:** "Connect a Facebook Page to review Messenger conversations alongside email, chat, and task tools. SlopWeaver helps surface what needs attention first and keeps the thread context nearby."

**Problem:** "Helps surface" is weak verb construction. "SlopWeaver helps surface" hedges the claim. Either it surfaces or it does not.

**Suggested fix:** "SlopWeaver surfaces what needs attention first and keeps thread context nearby."

---

### 8. GitHub -- "Full-Context Decisions" use case title

**File:** `_config.tsx`, github useCases
**Text:** "Full-Context Decisions"

**Problem:** Vague noun-phrase title. "Full-context" is used across multiple integrations as a filler modifier that means "you have more information." Not differentiated from "Better Context" (also used for GitHub).

---

### 9. Google Calendar -- "Stay Ahead of the Day" use case title

**File:** `_config.tsx`, google-calendar useCases
**Text:** "Stay Ahead of the Day"

**Problem:** Cliche productivity phrase. "Stay ahead of the day" appears in countless calendar apps, time-management blogs, and productivity tool marketing. No differentiation.

**Suggested fix:** "Know what is next without opening another tab."

---

### 10. Google Docs -- "Better Context" use case title (repeated pattern)

**File:** `_config.tsx`, google-docs useCases
**Text:** "Better Context"

**Problem:** "Better Context" is used as a use case title in multiple integrations (Google Docs, Jira, Linear, Microsoft Teams, Monday.com, Slack). It has become a filler title that could apply to any integration. The descriptions beneath are usually fine; the titles are generic placeholders.

**Suggested fix:** Make titles specific to the integration. For Google Docs: "Source material close at hand." For Jira: "Issue history without the tab switch." Etc.

---

## Notes on what is working

The integration template has strong structural decisions:

- "What your AI learns" callout per integration is well-differentiated and specific
- Security cards (OAuth-Only Access, Minimal Permissions, Revoke Anytime) are factual and reassuring without being overblown
- "Less Tab Hopping" / "Less Context Switching" use case titles are honest about the actual value proposition
- Technical details sections are appropriately factual (API names, OAuth flow description)
- The Facebook Messenger copy is notably more restrained than the Gmail/Asana copy -- it does not overpromise AI learning for a passive read-only feed

The biggest systemic issue is that the use case titles ("Better Context," "Full-Context Decisions," "Unified Communication," "Never Miss Deadlines") are generic across many integrations and do not carry their weight.
