# Trope Audit: apps/docs/src/data/integrations (Slice 135)

**Files audited:**

- `apps/docs/src/data/integrations/asana.tsx`
- `apps/docs/src/data/integrations/facebook-messenger.tsx`
- `apps/docs/src/data/integrations/github.tsx`
- `apps/docs/src/data/integrations/google-calendar.tsx`
- `apps/docs/src/data/integrations/google-docs.tsx`
- `apps/docs/src/data/integrations/google-drive.tsx`
- `apps/docs/src/data/integrations/google-gmail.tsx`
- `apps/docs/src/data/integrations/jira.tsx`

---

## asana.tsx

**intro:** "Connect Asana to track task activity in your inbox. SlopWeaver can suggest comments for your approval before anything is posted."

Clean. Concrete. No violations.

**subtitle:** "Import task and project activity into SlopWeaver"

Clean.

**syncCards descriptions:** Factual. No violations.

**No findings.**

---

## facebook-messenger.tsx

**intro:** "Connect a Facebook Page to manage inbox conversations and reply to messages. SlopWeaver syncs your page's Messenger conversations so you can handle customer messages alongside your other work."

**VIOLATION — "alongside your other work"**

This is mild AI-copy filler. It adds no information (of course it runs alongside other work; that is the premise of any app). The second sentence could simply end at "conversations."

**Suggested fix:** "Connect a Facebook Page to manage inbox conversations and reply to messages. SlopWeaver syncs your page's Messenger conversations into a single inbox."

**subtitle:** "Manage Facebook Page inbox conversations in SlopWeaver"

Clean.

**syncCards:** Factual. No violations.

**connectSteps:** Step 5 content: "SlopWeaver will begin syncing your page's conversations. The initial sync typically takes a few minutes depending on your message history. You'll see a progress indicator."

Clean documentation prose. No violations.

---

## github.tsx

**intro:** "Connect GitHub to track issue and pull request activity in your inbox. SlopWeaver can suggest comments for your approval before anything is posted."

Clean. Direct. No violations.

**subtitle:** "Import issue and pull request activity into SlopWeaver"

Clean.

**syncCards descriptions:** Factual. No violations.

**No findings.**

---

## google-calendar.tsx

**intro:** "Connect Google Calendar to surface meeting context in your inbox and assistant workflows. Read-only sync is available by default, and calendar changes only run after your approval."

**MILD FLAG — "surface meeting context"**

"Surface" as a verb is a common AI/product-writing cliche (meaning "make available" or "show"). Not a severe trope but worth watching across files. Used once here, acceptable in technical documentation context.

**subtitle:** "Import meetings and availability context into SlopWeaver"

Clean.

**connectStep 3:** "Nothing changes on your calendar unless you approve it from Queue."

Clean. Concrete reassurance.

**No hard violations. One minor style note flagged above.**

---

## google-docs.tsx

**intro:** "Connect Google Docs to sync and search your documents. SlopWeaver indexes your documents so your AI assistant can reference them and you can find what you need quickly."

**VIOLATION — "find what you need quickly"**

Vague filler. "Quickly" adds nothing. Every product claims speed. The sentence would be stronger without it.

**Suggested fix:** "...so your AI assistant can reference them and you can search across documents by content."

**subtitle:** "Sync and search your Google Docs in SlopWeaver"

Clean.

**syncCards:** Factual. No violations.

---

## google-drive.tsx

**intro:** "Connect Google Drive to index file content for assistant context and command search. Read-only is the default; read + write enables upload actions only when you approve."

Clean. Concrete. No violations.

**subtitle:** "Search and reference files from Drive across your workspace"

**MILD FLAG — "across your workspace"**

Vague filler phrase. Not a hard violation but "across your workspace" is padding. Could be trimmed: "Search and reference Drive files from SlopWeaver."

**syncCards:** Factual. No violations.

**No hard violations.**

---

## google-gmail.tsx

**intro:** "Connect your Gmail account to see your emails in the inbox. SlopWeaver syncs the messages you choose and suggested replies you review."

Clean. Direct. No violations.

**subtitle:** "Import your Gmail messages into SlopWeaver"

Clean.

**permissions entry (line 71):** "Only used when you explicitly approve and send a reply. Nothing sends without you."

Clean and direct. "Nothing sends without you" is a strong, clear statement.

**syncCards:** Factual. No violations.

**No findings.**

---

## jira.tsx

**intro:** "Connect Jira to track and manage issues from your inbox. SlopWeaver syncs your assigned issues, comments, and threads so you can stay on top of project work without switching tools."

**VIOLATION — "stay on top of" and "without switching tools"**

Both are stock productivity-software phrases.

- "Stay on top of" is an extremely overused expression in SaaS copy.
- "Without switching tools" / "without context-switching" is an equally worn productivity-app trope.

**Suggested fix:** "Connect Jira to track and manage issues from your inbox. SlopWeaver syncs assigned issues, comments, and threads so they appear alongside your email and chat."

**subtitle:** "Track and manage Jira issues in SlopWeaver"

Clean.

**syncCards:** Factual. No violations.

---

## Summary of Violations (Slice 135)

| File                   | Violation                                                           | Severity |
| ---------------------- | ------------------------------------------------------------------- | -------- |
| facebook-messenger.tsx | "alongside your other work" — filler                                | Low      |
| google-docs.tsx        | "find what you need quickly" — vague filler                         | Low      |
| google-drive.tsx       | "across your workspace" — padding                                   | Low      |
| jira.tsx               | "stay on top of" and "without switching tools" — dual stock phrases | Medium   |
| google-calendar.tsx    | "surface" as a verb — mild style note                               | Minimal  |
