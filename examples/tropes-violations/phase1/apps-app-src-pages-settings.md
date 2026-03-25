# Trope Audit: apps/app/src/pages/settings

Files audited:

- `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/settings/page.tsx`

---

## page.tsx

### User-facing strings inventory

Section titles and descriptions are defined in the `sectionTitles` record:

| Section           | Title               | Description                              |
| ----------------- | ------------------- | ---------------------------------------- |
| account           | "Account"           | "Manage security and data"               |
| billing           | "Billing & Usage"   | "Manage usage and subscription"          |
| diagnostics       | "Diagnostics"       | "Export logs and test connectivity"      |
| integrations      | "Integrations"      | "Connect and configure your platforms"   |
| knowledge-sources | "Knowledge Sources" | "Extra context for better AI responses"  |
| profile           | "Profile"           | "Your profile, built from your activity" |
| rules             | "Rules"             | "Define deterministic AI rules"          |

Additional UI strings:

| Location                                 | Text                                       |
| ---------------------------------------- | ------------------------------------------ |
| Save status                              | "Saving"                                   |
| Save status                              | "Saved"                                    |
| Mobile header aria-label                 | "Open settings menu"                       |
| Mobile logout aria-label                 | "Log Out"                                  |
| Toast (reconnect success, with provider) | "{Provider} reconnected"                   |
| Toast (reconnect success, generic)       | "Integration reconnected"                  |
| Toast (reconnect) description            | "Tokens have been refreshed successfully." |

### Analysis

Examining each section description:

- **"Manage security and data"** (Account): Direct. No trope.
- **"Manage usage and subscription"** (Billing): Direct. No trope.
- **"Export logs and test connectivity"** (Diagnostics): Direct, functional. No trope.
- **"Connect and configure your platforms"** (Integrations): Direct. No trope.
- **"Extra context for better AI responses"** (Knowledge Sources): Uses "better AI responses" which is slightly generic but is describing a concrete mechanism (adding context improves output). Borderline but acceptable given it describes what the feature actually does.
- **"Your profile, built from your activity"** (Profile): Concrete, explains the mechanism. No trope.
- **"Define deterministic AI rules"** (Rules): The word "deterministic" is technically precise and differentiating. No trope.

Toast copy is functional and plain.

**Findings:** None requiring change. The section descriptions are concise and functional.
