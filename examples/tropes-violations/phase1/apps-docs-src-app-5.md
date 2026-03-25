# Tropes Audit: apps/docs/src/app — Slice 5

**Files audited** (remaining integration pages):

- `src/app/integrations/google-calendar/page.tsx`
- `src/app/integrations/google-docs/page.tsx`
- `src/app/integrations/google-drive/page.tsx`
- `src/app/integrations/microsoft-calendar/page.tsx`
- `src/app/integrations/microsoft-onedrive/page.tsx`
- `src/app/integrations/microsoft-teams/page.tsx`
- `src/app/integrations/monday/page.tsx`
- `src/app/integrations/facebook-messenger/page.tsx`

---

## Structure note

All 8 files follow the same `IntegrationPageTemplate` wrapper pattern as Slice 4. Only the SEO metadata `description` fields contain user-visible text in these files.

---

## Metadata descriptions audited

**Google Calendar**:

> "Connect Google Calendar to sync events, attendees, locations, and availability. Powers meeting prep briefings and conflict detection for scheduling suggestions."

**Google Docs**:

> "Connect Google Docs to sync and search your documents. SlopWeaver indexes document content so the AI assistant can reference it when drafting replies and preparing briefings."

**Google Drive**:

> "Connect Google Drive to index PDFs, documents, spreadsheets, and other files for AI context and search. Read-only by default with optional upload actions via approval."

**Microsoft Calendar**:

> "Connect Microsoft Calendar to sync events, Teams links, attendees, and availability. Read-only by default with optional staged event actions via the Queue."

**Microsoft OneDrive**:

> "Connect Microsoft OneDrive to index file content for AI assistant context and search. Read-only by default with optional upload actions requiring your approval."

**Microsoft Teams**:

> "Connect Microsoft Teams to sync chat conversations into SlopWeaver. Unify Teams chat triage with email and project management workflows, with queued reply actions for approval."

**Monday.com**:

> "Connect Monday.com to sync board items and updates into SlopWeaver. Manage project work from your inbox without context-switching between tools."

**Facebook Messenger**:

> "Connect a Facebook Page to manage Messenger inbox conversations in SlopWeaver. Sync page messages and handle customer replies alongside your other work."

---

## Findings

**Microsoft Teams metadata**: "Unify Teams chat triage with email and project management workflows" — "Unify" is a mild marketing verb. The description is accurate but could be more direct: "Sync Teams chat alongside email and project management."

**Monday.com metadata**: "without context-switching between tools" — negative parallelism defining the benefit by what the user avoids. "Manage project work from your inbox" already makes the point; the appended clause is redundant.

All other metadata descriptions are factual and free of significant flagged tropes.

**Severity**: Minimal.

---

## Summary for Slice 5

| File                        | Findings                                     |
| --------------------------- | -------------------------------------------- |
| google-calendar/page.tsx    | None                                         |
| google-docs/page.tsx        | None                                         |
| google-drive/page.tsx       | None                                         |
| microsoft-calendar/page.tsx | None                                         |
| microsoft-onedrive/page.tsx | None                                         |
| microsoft-teams/page.tsx    | 1 minimal (mild marketing verb)              |
| monday/page.tsx             | 1 minimal (negative parallelism in metadata) |
| facebook-messenger/page.tsx | None                                         |

**Note**: As in Slice 4, the actual page body content is rendered by `IntegrationPageTemplate` from config data. The config files are not in scope for this slice.
