# Tropes Audit: apps/app/src/components/platform-content

Files audited:

- `platform-content-renderer.tsx`

## Findings

### 1. "Platform content not available for preview"

**File**: `platform-content-renderer.tsx`, line 473
**Category**: Evasive AI fallback copy

**Text**:

```
"Platform content not available for preview"
```

**Problem**: Passive, vague fallback message. It describes an internal system state ("not available for preview") rather than telling the user what happened or what to do. "Preview" is a product-internal concept that leaks into the UI. A user does not think of this view as a "preview" - they just want to read the content.

**Suggested fix**: Something concrete and action-oriented, e.g., "Content couldn't be loaded" with an optional "Open original" link (which already exists in the same block when `fallback.url` is present). The fix is to rely solely on the link when available, and drop the passive fallback text or replace it with nothing when there is nothing useful to say.

---

No other AI writing tropes found. The file is primarily dispatch logic and prop plumbing with no other user-facing copy.
