# Trope Audit: apps/app/src/pages/triage

Files audited:

- `page.tsx`

## Findings

### page.tsx

**"You've used all your trial actions."** (line 355)
No issue. Direct statement of account state.

**"Upgrade to continue, or skip remaining items to see your results."** (line 357)
No issue. Direct instruction with two concrete options.

**"Upgrade"** CTA (line 362) / **"Skip remaining"** button (line 366)
No issue. Direct action labels.

**"Session started"** / platform list in header (line 316)
No issue. Direct session state description.

**`{Math.round(...)}% complete`** progress display (line 322)
No issue. Numeric progress, not copy.

**"Triage"** page title (line 314)
No issue. Direct feature name.

No AI-flavored copy, hype language, vague benefit statements, or other writing tropes found. The page's user-visible text is limited to session state, a trial exhaustion warning, and action labels -- all concrete and direct.

## Summary

| File     | Violations |
| -------- | ---------- |
| page.tsx | 0          |

No findings.
