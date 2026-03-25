# Tropes Audit: apps/docs/src/app — Slice 4

**Files audited** (integration pages that use the `IntegrationPageTemplate` component):

- `src/app/integrations/google-gmail/page.tsx`
- `src/app/integrations/slack/page.tsx`
- `src/app/integrations/linear/page.tsx`
- `src/app/integrations/microsoft-outlook/page.tsx`
- `src/app/integrations/asana/page.tsx`
- `src/app/integrations/github/page.tsx`
- `src/app/integrations/jira/page.tsx`
- `src/app/integrations/notion/page.tsx`

---

## Structure note

All 8 of these files follow an identical pattern: they are thin wrapper components that render `<IntegrationPageTemplate config={...} />`. The only user-facing text in these files is in the SEO metadata `description` field. No prose body content exists in these files themselves — the actual page content is rendered by the template from config objects (not audited here).

---

## Metadata descriptions audited

**Gmail**:

> "Connect your Gmail account to sync email threads, labels, and attachments. Read-only by default with optional send permissions for approved AI-drafted replies."

**Slack**:

> "Connect Slack workspaces to sync DMs, mentions, and threads into your inbox. Supports multiple workspaces with adjustable sync focus from direct involvement to broad channel context."

**Linear**:

> "Connect Linear to sync assigned issues, comments, and metadata with two-way sync. Status changes, comments, and priority updates flow bidirectionally between SlopWeaver and Linear."

**Outlook**:

> "Connect Microsoft Outlook to sync email messages and conversation context. Queue reply, archive, and mark-as-read actions for approval alongside your Slack and Linear workflows."

**Asana**:

> "Connect Asana to sync task activity, project metadata, and comments into SlopWeaver. AI can suggest task comments for your approval before posting."

**GitHub**:

> "Connect GitHub to track issue and pull request notifications in your inbox. AI can suggest comments on issues and PRs for your approval before posting."

**Jira**:

> "Connect Jira to sync assigned issues, comments, and project threads into SlopWeaver. Track and manage Jira work from your inbox without switching tools."

**Notion**:

> "Connect Notion to sync and search your workspace pages. SlopWeaver indexes Notion content so the AI assistant can reference it across all your connected tools."

---

## Findings

**Linear metadata**: "Status changes, comments, and priority updates flow bidirectionally between SlopWeaver and Linear." — "flow bidirectionally" is moderately ornate. "sync both ways" or "sync in both directions" would be direct.

All other metadata descriptions are factual, direct, and free of flagged tropes.

**Severity**: Minimal. One minor phrasing issue in one metadata description.

---

## Summary for Slice 4

| File                       | Findings                                   |
| -------------------------- | ------------------------------------------ |
| google-gmail/page.tsx      | None                                       |
| slack/page.tsx             | None                                       |
| linear/page.tsx            | 1 minimal (ornate verb phrase in metadata) |
| microsoft-outlook/page.tsx | None                                       |
| asana/page.tsx             | None                                       |
| github/page.tsx            | None                                       |
| jira/page.tsx              | None                                       |
| notion/page.tsx            | None                                       |

**Note**: The bulk of content for these pages lives in the config data files (`src/data/integrations/*.ts`) and the `IntegrationPageTemplate` component, which are not included in this slice. If those files contain user-facing prose, they should be audited separately.
