# Trope Audit: apps/docs/src/data/integrations (Slice 136)

**Files audited:**

- `apps/docs/src/data/integrations/linear.tsx`
- `apps/docs/src/data/integrations/linkedin.tsx`
- `apps/docs/src/data/integrations/microsoft-calendar.tsx`
- `apps/docs/src/data/integrations/microsoft-onedrive.tsx`
- `apps/docs/src/data/integrations/microsoft-outlook.tsx`
- `apps/docs/src/data/integrations/microsoft-teams.tsx`
- `apps/docs/src/data/integrations/monday.tsx`
- `apps/docs/src/data/integrations/notion.tsx`

---

## linear.tsx

**intro:** "Connect your Linear workspace to sync issues, comments, and assignments. SlopWeaver tracks issue updates as tasks and enables two-way sync for status changes."

Clean. Concrete. No violations.

**subtitle:** "Import your Linear workspace into SlopWeaver"

Clean.

**syncCards:** Factual. No violations.

**syncExtra callout:** "Adjust Sync Focus in Settings to control breadth. Focused prioritizes issues involving you; Broad widens to more team context."

Clean documentation prose. No violations.

**Two-Way Sync extraSections prose:** "SlopWeaver syncs changes bidirectionally. Updates in SlopWeaver flow back to Linear, and vice versa."

Clean.

**No findings.**

---

## linkedin.tsx

**intro:** "Connect your LinkedIn account to create posts directly from SlopWeaver. Your AI assistant can help draft LinkedIn content in your voice."

**VIOLATION — "in your voice"**

"In your voice" is one of the most overused AI-product phrases. It appears constantly in AI writing tools and has become meaningless through repetition. This is especially ironic because SlopWeaver's own core positioning is about voice/learning, so this phrase is both a trope AND a missed opportunity to say something more specific.

**Suggested fix:** "Connect your LinkedIn account to create posts from SlopWeaver. The assistant drafts based on your past writing, which you review and approve before publishing."

**subtitle:** "Create posts on LinkedIn from SlopWeaver"

Clean.

**syncExtra callout (Action-Only Integration):** "LinkedIn is an action-only integration. It does not sync your feed or messages. Use it to create and interact with content through SlopWeaver's AI assistant."

**MILD FLAG — "interact with content"**

Vague. LinkedIn is write-only (post creation). "Interact with content" implies more than what's available. Could be simplified to: "Use it to draft and publish posts via the assistant."

**connectStep 4:** "Once connected, you can use the AI assistant to draft LinkedIn posts. All posts require your explicit approval before being published."

Clean. Concrete. No violations.

---

## microsoft-calendar.tsx

**intro:** "Connect Microsoft Calendar to surface meeting context in your inbox and assistant workflows. Read-only sync is available by default, and calendar changes only run after your approval."

**MILD FLAG — "surface meeting context"**

Same pattern as google-calendar.tsx. "Surface" as a verb is a common product-writing cliche. Appears in both calendar files — consistent but still worth fixing.

**subtitle:** "Bring meetings and availability into your SlopWeaver console"

**MILD FLAG — "console"**

The product is not described as a "console" elsewhere in audited files. Inconsistent terminology. Could align with "inbox" which is used consistently elsewhere.

**syncCards:** Factual. No violations.

**No hard violations. Two minor style notes.**

---

## microsoft-onedrive.tsx

**intro:** "Connect OneDrive to index file content for assistant context and command search. Read-only is the default; read + write enables upload actions only when you approve."

Clean. Mirrors google-drive.tsx intentionally. No violations.

**subtitle:** "Search and reference files from OneDrive across your workspace"

**MILD FLAG — "across your workspace"**

Same phrase flagged in google-drive.tsx. Padding. Could be "Search and reference OneDrive files from SlopWeaver."

**syncCards:** Factual. No violations.

**No hard violations.**

---

## microsoft-outlook.tsx

**intro:** "Connect Outlook to unify email triage with Slack and Linear workflows. SlopWeaver syncs messages, keeps conversation context, and lets you queue archive/mark-read/reply actions for approval."

**Assessment:** This is the most specific and concrete intro of all the integration files. "Unify email triage with Slack and Linear workflows" says something real. "Keeps conversation context" is product-specific. No violations.

**subtitle:** "Import your Outlook inbox into SlopWeaver"

Clean.

**syncCards:** Factual. No violations.

**No findings.**

---

## microsoft-teams.tsx

**intro:** "Connect Teams to unify chat triage with email and project management workflows. SlopWeaver syncs chats, keeps conversation context, and lets you queue reply actions for approval."

Clean. Mirrors Outlook structure appropriately. No violations.

**subtitle:** "Import your Teams chats into SlopWeaver"

Clean.

**syncCards:** Factual. No violations.

**No findings.**

---

## monday.tsx

**intro:** "Connect Monday.com to track and manage board items from your inbox. SlopWeaver syncs your items and updates so you can manage project work without context-switching."

**VIOLATION — "without context-switching"**

Same trope flagged in jira.tsx. This is a stock SaaS productivity phrase that appears throughout the industry and adds no information.

**Suggested fix:** "Connect Monday.com to track and manage board items from your inbox. SlopWeaver syncs board items and updates alongside your email and chat."

**subtitle:** "Track and manage Monday.com items in SlopWeaver"

Clean.

**syncCards:** Factual. No violations.

---

## notion.tsx

**intro:** "Connect Notion to sync and search your workspace pages. SlopWeaver indexes your Notion content so your AI assistant can reference it and you can find what you need across all your tools."

**VIOLATION — "find what you need across all your tools"**

Two problems in one phrase:

1. "Find what you need" is vague filler (same pattern as google-docs.tsx).
2. "Across all your tools" is a classic SaaS marketing phrase implying unified-everything, which overstates what Notion sync alone provides.

**Suggested fix:** "Connect Notion to sync and search your workspace pages. SlopWeaver indexes Notion content so the assistant can reference it and you can search across pages."

**subtitle:** "Sync and search your Notion pages in SlopWeaver"

Clean.

**syncCards:** Factual. No violations.

---

## Summary of Violations (Slice 136)

| File                   | Violation                                                | Severity |
| ---------------------- | -------------------------------------------------------- | -------- |
| linkedin.tsx           | "in your voice" — overused AI trope                      | High     |
| linkedin.tsx           | "interact with content" — vague                          | Low      |
| microsoft-calendar.tsx | "surface meeting context" — mild style                   | Minimal  |
| microsoft-calendar.tsx | "console" — inconsistent terminology                     | Low      |
| microsoft-onedrive.tsx | "across your workspace" — padding                        | Low      |
| monday.tsx             | "without context-switching" — stock SaaS phrase          | Medium   |
| notion.tsx             | "find what you need across all your tools" — dual filler | Medium   |
