# Trope Audit: apps/marketing/src/app/integrations/ (Part 2)

**Slice:** 145
**Files audited:** 8

- `integrations/google-drive/page.tsx`
- `integrations/google-gmail/page.tsx`
- `integrations/jira/page.tsx`
- `integrations/linear/page.tsx`
- `integrations/linkedin/page.tsx`
- `integrations/microsoft-calendar/page.tsx`
- `integrations/microsoft-onedrive/page.tsx`
- `integrations/microsoft-outlook/page.tsx`

All copy lives in `_config.tsx`. Individual `page.tsx` files are thin wrappers. Template-level violations documented in Part 1 (slice 144) are not repeated here; only integration-specific copy is audited.

---

## Summary

Moderate violations. Gmail has the most marketing-forward copy of all integrations and shows the most trope risk. Most other integrations in this slice are more restrained. The "Better Context" and "Never Miss Deadlines" / "Stay Ahead of the Day" patterns documented in Part 1 recur here.

---

## Violations

### 1. Gmail -- heroDescription feature stacking

**File:** `_config.tsx`, google-gmail heroDescription
**Text:** "Connect Gmail and the AI starts learning your email patterns. Response timing, tone per recipient, thread priority. Track the improvement with acceptance rate and time-saved metrics."

**Problem:** The second sentence ("Response timing, tone per recipient, thread priority") is a comma-stacked feature list in fragment form. The third sentence refers to "the improvement" before establishing there is anything to improve. The sequence implies a certainty of improvement rather than a possibility. Minor issue -- the copy is mostly good -- but "Track the improvement" assumes outcome.

**Suggested fix:** "Connect Gmail and the AI starts learning your email patterns: response timing, tone per recipient, thread priority. Open analytics after a week and see how the acceptance rate changed."

---

### 2. Gmail -- "Never Miss Deadlines" use case title (same as Asana)

**File:** `_config.tsx`, google-gmail useCases
**Text:** "Never Miss Deadlines"

**Problem:** Same generic productivity promise documented in Part 1 for Asana. The description ("Action items extracted from email become tasks with due dates. No more hunting through threads to find what you promised.") is better than the title.

**Suggested fix:** Use the stronger description as the basis: "Action items out of threads" or "Follow-up extraction."

---

### 3. Gmail -- "Unified Communication" use case title (same as Asana)

**File:** `_config.tsx`, google-gmail useCases
**Text:** "Unified Communication"

**Problem:** Same issue as Asana slice. "Unified Communication" is enterprise telecom jargon, not a use case description for a Gmail integration. The description ("Handle Gmail alongside Slack and Linear in one view") is correct; the title does not match the description's register.

---

### 4. Jira -- "Better Context" use case title

**File:** `_config.tsx`, jira useCases
**Text:** "Better Context"

**Problem:** Repeated generic title. See Part 1 pattern note. This is the third or fourth instance of "Better Context" as a use case title across integrations.

---

### 5. Linear -- "Better Context" use case title

**File:** `_config.tsx`, linear useCases
**Text:** "Better Context"

**Problem:** Same as Jira. Fifth or more instance of this title across the integration pages.

---

### 6. LinkedIn -- "Write with Context" use case title

**File:** `_config.tsx`, linkedin useCases
**Text:** "Write with Context"

**Problem:** Weak, generic title. "Write with context" could mean anything. The description ("Keep launch or campaign context close while refining the final post") is clearer.

**Suggested fix:** "Draft from your notes and launch plans" or "Campaign context stays close."

---

### 7. LinkedIn -- heroDescription passive hedging

**File:** `_config.tsx`, linkedin heroDescription
**Text:** "Connect LinkedIn to draft posts alongside your notes, inbox threads, and launch planning. SlopWeaver helps you shape copy in context, then keeps publishing behind explicit approval."

**Problem:** "Helps you shape copy" is weak-verb hedging. Either SlopWeaver shapes the copy (with you reviewing) or it does not. "Shape copy in context" is also awkward.

**Suggested fix:** "Connect LinkedIn to draft posts from your notes and launch plans. You review, then approve before anything publishes."

---

### 8. Microsoft Calendar -- "Stay Ahead of the Day" use case title (same as Google Calendar)

**File:** `_config.tsx`, microsoft-calendar useCases
**Text:** "Stay Ahead of the Day"

**Problem:** Exact duplicate of the Google Calendar use case title. Cliche productivity phrase. Both Calendar integrations use identical section titles ("Prepare Faster," "Follow Through After Calls," "Stay Ahead of the Day," "One Operating View"), which makes the pages feel templated rather than platform-specific.

---

### 9. Microsoft Outlook -- "Connected Workflow" use case title

**File:** `_config.tsx`, microsoft-outlook useCases
**Text:** "Connected Workflow"

**Problem:** "Connected workflow" is generic productivity jargon used across the entire SaaS industry. It describes no specific outcome. The description ("Handle mailbox work that depends on meetings, chats, or project updates from one place") is adequate; the title is not.

**Suggested fix:** "Mail alongside meetings and tasks" or "One view for Outlook, Teams, and Calendar."

---

## Notes on what is working

- LinkedIn copy is notably honest about what the integration does (action-only, explicit publish approval). "Nothing goes live until you review and approve it" is direct and trustworthy.
- OneDrive and Google Drive use almost identical copy structures (both are file storage integrations). This is appropriate and not a trope violation -- they genuinely do the same thing.
- Outlook's "Respond with Context" use case description ("Keep thread context close while reviewing follow-ups across mail, chat, and tasks") is specific and accurate.
- Microsoft Calendar and Google Calendar having near-identical copy is architecturally honest -- they provide equivalent functionality. It is not a trope violation to have parallel structure.
