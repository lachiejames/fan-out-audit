# Trope Audit: apps/app/src/shared/platforms

Files audited:

- `platform-icons.tsx`

## Findings

### platform-icons.tsx

No user-visible copy in this file. The file contains:

- Utility functions for resolving platform brand colors, badge colors, and icon components from the contracts registry
- A hex-to-rgba conversion helper
- A `getPlatformLabelUnsafe` helper that delegates to `getPlatformDisplayName` from contracts

All strings in the file are internal: JSDoc comments, error fallback values (`rgba(255,255,255,${alpha})`), and type annotations. None are rendered to the UI by this file directly.

No tropes, hype language, or AI writing patterns found.

## Summary

| File               | Violations |
| ------------------ | ---------- |
| platform-icons.tsx | 0          |

No findings.
