# Type-Safety Audit: apps-app-src-components-22

**Files audited:**

- `apps/app/src/components/settings/knowledge-sources/knowledge-source-snapshot-provider.tsx`
- `apps/app/src/components/settings/knowledge-sources/knowledge-source-template-presets.ts`
- `apps/app/src/components/settings/knowledge-sources/knowledge-source-upload-card.tsx`
- `apps/app/src/components/settings/knowledge-sources/knowledge-sources-empty-state.tsx`
- `apps/app/src/components/settings/knowledge-sources/searchable-archive-section.tsx`
- `apps/app/src/components/settings/profile/settings-ai-context-modal.tsx`
- `apps/app/src/components/settings/profile/settings-ai-snapshot-card.tsx`
- `apps/app/src/components/settings/profile/settings-basics-overrides-section.tsx`

---

## Findings

### 1. `Record<string, string>` with known key domain — `searchable-archive-section.tsx` line 23

**Category:** `Record<string, ...>` where stricter types would be safer

**Location:** `apps/app/src/components/settings/knowledge-sources/searchable-archive-section.tsx:23`

**Description:**
`PHASE_LABELS` is declared as `Record<string, string>`. The keys are phase identifiers that are sourced from the contract's `SyncPhase` union type. Using `Record<string, string>` means any string key is accepted at read time, losing the exhaustiveness guarantee that the contract provides.

**Current code (approximate):**

```typescript
const PHASE_LABELS: Record<string, string> = {
  discovery: "Discovery",
  // ...
};
```

**Suggested fix:**

```typescript
import type { SyncPhase } from "@slopweaver/contracts";
const PHASE_LABELS: Record<SyncPhase, string> = { ... };
```

This ensures a compile-time error if a new `SyncPhase` value is added without a corresponding label.

---

### 2. `Object.entries` on loosely typed `categoryCounts` — `settings-ai-context-modal.tsx`

**Category:** `Record<string, ...>` where stricter types would be safer

**Location:** `apps/app/src/components/settings/profile/settings-ai-context-modal.tsx`

**Description:**
`source.importSummary.categoryCounts` is iterated with `Object.entries(...)`. If the contract defines `categoryCounts` as `Record<string, number>`, that's acceptable. However, if the field comes from a discriminated response type or a schema that allows `unknown` values, the iteration loses type safety. Verify that `categoryCounts` in `@slopweaver/contracts` is typed as `Record<string, number>` (not `Record<string, unknown>` or `object`) and that the entries are safe to use directly without narrowing.

**No code change required** if the contract already types this as `Record<string, number>`. This is a verification note.

---

## Clean Files

- `knowledge-source-snapshot-provider.tsx` — No issues found.
- `knowledge-source-template-presets.ts` — Well-typed preset array; no issues.
- `knowledge-source-upload-card.tsx` — No issues found.
- `knowledge-sources-empty-state.tsx` — No issues found.
- `settings-ai-snapshot-card.tsx` — Correctly uses indexed access types (`CoreProfileResponse["identity"]`); no issues.
- `settings-basics-overrides-section.tsx` — No issues found.
