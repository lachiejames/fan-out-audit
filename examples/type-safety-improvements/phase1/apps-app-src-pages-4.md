# Type-Safety Audit: apps-app-src-pages-4

**Files inspected**: 8
**Findings**: 3

## Summary

`sync-stream-ui.utils.ts` has a cast on a `LearningEventType` that could be replaced with a narrowing check. `queue/page.tsx` uses `Record<string, number>` for platform counts where `Partial<Record<PlatformId, number>>` would be safer. The `settings/page.tsx` has a `tabFromUrl as SettingsSection | null` cast that could be replaced with a `Set.has` check. Auth pages (`login/page.tsx`, `reset-password/page.tsx`) and onboarding files are clean (auth page casts already reported in pages-2).

---

## Findings

### 1

**File**: `apps/app/src/pages/integrations/_shared/sync-stream-ui.utils.ts`
**Category**: Type casts (`as SomeType`) that could be avoided
**Impact**: Low — the cast is on a runtime value that has already been validated by a schema at the knowledge-extraction layer
**Description**: `fact.category as LearningEventType` at line 211 casts the raw category string. The `SyncStreamKnowledgeExtractedEvent` type (from contracts) already types `fact.category` as `string`. If `LearningEventType` is a union derived from the same values as `KnowledgeCategory`, a type guard or an explicit assertion with an exhaustive check would be safer.
**Suggestion**: Add `isLearningEventType(v: string): v is LearningEventType` and call it before the cast. If the cast would always succeed (because `LearningEventType` is identical to `KnowledgeCategory`), import `LearningEventType` directly from contracts and validate at the boundary.
**Evidence**: `type: fact.category as LearningEventType` — line 211

---

### 2

**File**: `apps/app/src/pages/queue/page.tsx`
**Category**: `Record<string, ...>` where stricter types exist
**Impact**: Low — platform IDs are dynamic at runtime so a `string` key is pragmatic; but `PlatformId` is available
**Description**: `filteredPlatformCounts: Record<string, number>` is built from `item.platform` on lines 183–188. The `QueueItem.platform` field is typed as `string` in the component type, but `PlatformId` is available from contracts.
**Suggestion**: If `QueueItem.platform` can be narrowed to `PlatformId`, change the accumulator type to `Partial<Record<PlatformId, number>>`. If `QueueItem.platform` is intentionally `string`, this finding is not actionable until the queue item type is tightened.
**Evidence**:

```typescript
const counts: Record<string, number> = {};
for (const item of typeConfidenceFilteredItems) {
  counts[item.platform] = (counts[item.platform] ?? 0) + 1;
}
```

---

### 3

**File**: `apps/app/src/pages/settings/page.tsx`
**Category**: Type casts (`as SomeType`) that could be avoided
**Impact**: Low — the cast is guarded by `VALID_SECTIONS.has(tabFromUrl)` a line later, so it is safe; but the intermediate variable is of the cast type
**Description**: `const tabFromUrl = searchParams.get("tab") as SettingsSection | null` at line 58 casts a raw URL string. The actual safety check happens on line 59 via `VALID_SECTIONS.has(tabFromUrl)`. The cast on line 58 is premature — it asserts a type before the guard has run.
**Suggestion**: Keep `tabFromUrl` as `string | null`, then derive `activeSection` inside the `Set.has` check:

```typescript
const tabFromUrl = searchParams.get("tab");
const activeSection: SettingsSection =
  tabFromUrl !== null && VALID_SECTIONS.has(tabFromUrl as SettingsSection)
    ? (tabFromUrl as SettingsSection)
    : "integrations";
```

Or extract `isSettingsSection(v: string): v is SettingsSection` as a type predicate.
**Evidence**: `const tabFromUrl = searchParams.get("tab") as SettingsSection | null;` — line 58
