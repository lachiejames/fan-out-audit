# Audit: apps-app-src-components-18

**Files inspected**:

- `apps/app/src/components/queue/queue-audit-detail-renderer.tsx`
- `apps/app/src/components/queue/queue-audit-log-view.tsx`
- `apps/app/src/components/queue/queue-card.tsx`
- `apps/app/src/components/queue/queue-empty-state.tsx`
- `apps/app/src/components/queue/queue-filter-bar.tsx`
- `apps/app/src/components/queue/queue-filter-options.ts`
- `apps/app/src/components/queue/queue-origin-metadata.ts`
- `apps/app/src/components/queue/queue-preview-badge.utils.ts`

**Findings**: 3

---

## Summary

Three findings. `DETAIL_FIELD_LABELS` in `queue-audit-detail-renderer.tsx` should use keys derived from `KNOWN_DETAIL_FIELDS` rather than `string`. `WORK_ITEM_TYPE_LABELS` in `queue-filter-options.ts` should use `WorkItemType` keys rather than `string`. `TERMINAL_STATUS_BADGES` in `queue-preview-badge.utils.ts` should use `WorkItemStatus` keys. All three are `Record<string, string>` maps where the valid key set is already defined elsewhere and could be encoded in the type.

---

## Findings

### Finding 1: `DETAIL_FIELD_LABELS: Record<string, string>` should be keyed by `KNOWN_DETAIL_FIELDS` members

- **File**: `apps/app/src/components/queue/queue-audit-detail-renderer.tsx`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `KNOWN_DETAIL_FIELDS` is a `readonly` tuple of known field names at line 9. `DETAIL_FIELD_LABELS` at line 22 is typed `Record<string, string>`, accepting any string key. Using `Partial<Record<(typeof KNOWN_DETAIL_FIELDS)[number], string>>` would make the two structures consistent — any entry in `DETAIL_FIELD_LABELS` must be a known field name, and TypeScript will flag misspelled or removed keys.
- **Suggestion**: Change the type annotation to `Partial<Record<(typeof KNOWN_DETAIL_FIELDS)[number], string>>`.
- **Evidence**:
  ```typescript
  // line 9
  const KNOWN_DETAIL_FIELDS = ["statusFrom", "statusTo", "platform", ...] as const;
  // line 22
  const DETAIL_FIELD_LABELS: Record<string, string> = { ... }
  // Should be: Partial<Record<(typeof KNOWN_DETAIL_FIELDS)[number], string>>
  ```

---

### Finding 2: `WORK_ITEM_TYPE_LABELS: Record<string, string>` should use `WorkItemType` keys

- **File**: `apps/app/src/components/queue/queue-filter-options.ts`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: `WORK_ITEM_TYPE_LABELS` at line 15 is `Record<string, string>`. The keys are `WorkItemType` values from `@slopweaver/contracts`. Using `Partial<Record<WorkItemType, string>>` would ensure that when a new work item type is added to contracts, the label map can be updated safely — any key not in `WorkItemType` becomes a compile error.
- **Suggestion**: Change the type annotation to `Partial<Record<WorkItemType, string>>` and import `WorkItemType` from `@slopweaver/contracts`.
- **Evidence**:
  ```typescript
  // line 15
  const WORK_ITEM_TYPE_LABELS: Record<string, string> = {
    add_reaction: "Reaction",
    send_reply: "Reply",
    ...
  }
  ```

---

### Finding 3: `TERMINAL_STATUS_BADGES: Record<string, string>` should use `WorkItemStatus` keys

- **File**: `apps/app/src/components/queue/queue-preview-badge.utils.ts`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `TERMINAL_STATUS_BADGES` at line 5 is `Record<string, string>`. The keys are a subset of `WorkItemStatus` terminal states. Using `Partial<Record<WorkItemStatus, string>>` would make the type-safe against contract changes and allow exhaustiveness checking.
- **Suggestion**: Change to `Partial<Record<WorkItemStatus, string>>` and import `WorkItemStatus` from `@slopweaver/contracts`.
- **Evidence**:
  ```typescript
  // line 5
  const TERMINAL_STATUS_BADGES: Record<string, string> = {
    discarded: "Discarded",
    expired: "Expired",
    failed: "Failed",
    undone: "Undone",
  };
  ```
