# Tropes Audit: apps/app/src/components/integrations (Slice 86)

Files audited:

- connection-wizard/stepper-icon.tsx
- core-integration-card.tsx
- directory-view.tsx
- integration-catalog.tsx
- integration-connect-modal.tsx
- integration-connection-row.tsx
- integration-row.tsx

---

## Violations

### 1. "Email threads + suggested replies" descriptions (×2)

**File**: `core-integration-card.tsx`
**Lines**: 23, 29
**Offending text**:

```typescript
"google-gmail": "Email threads + suggested replies",
"microsoft-outlook": "Email threads + suggested replies",
```

**Trope**: "Suggested replies" as a copy fragment reads as a feature-list talking point rather than a user benefit description. Identical copy for two different platforms is lazy and reduces distinctiveness. The phrase "suggested replies" is also an AI-product cliche.
**Fix**: Write distinct descriptions for Gmail and Outlook that say what the integration actually does in concrete terms, e.g., "Gmail threads, labels, and drafts" / "Outlook inbox, folders, and calendar events".

---

### 2. "Connect your tools" — vague onboarding header

**File**: `directory-view.tsx`
**Line**: 92
**Offending text**:

```tsx
<h2 className="text-base font-semibold text-foreground sm:text-lg">Connect your tools</h2>
<p className="text-xs text-foreground/60 mt-0.5 sm:text-sm">Choose what to sync</p>
```

**Trope**: "Connect your tools" is generic AI-product onboarding copy. Every productivity SaaS uses this exact heading. "Choose what to sync" as a subtitle is equally undifferentiated.
**Fix**: Use copy that reflects SlopWeaver's positioning. For example: "Connect your work accounts" with a subtitle that references what SlopWeaver actually does with the data, e.g., "SlopWeaver reads these to learn your patterns and draft replies."

---

### 3. "Connect your tools" section label (settings variant)

**File**: `directory-view.tsx`
**Line**: 143
**Offending text**:

```tsx
{
  connectedCount > 0 ? "Your integrations" : "Connect your tools";
}
```

**Trope**: Same "Connect your tools" copy repeated as a section label in the non-onboarding settings view. Same issue as violation 2.
**Fix**: Replace the empty-state label with something more specific, e.g., "No integrations connected" or simply "Available integrations".

---

### 4. "Connect your [platform] account to start syncing"

**File**: `integration-connect-modal.tsx`
**Line**: 154
**Offending text**:

```tsx
<p className="text-sm text-muted-foreground">Connect your {integration.name} account to start syncing</p>
```

**Trope**: "To start syncing" is a filler purpose clause. The button immediately below says "Connect" — the inline text adds nothing. This phrasing pattern ("connect X to start Y") is ubiquitous in AI product onboarding and adds no information.
**Fix**: Either remove this empty-state text (the Connect button is self-explanatory) or replace with a single concrete sentence describing what the integration enables for this specific platform.

---

### 5. "Docs + files in one searchable stream" — "stream" jargon

**File**: `core-integration-card.tsx`
**Line**: 22
**Offending text**:

```typescript
"google-drive": "Docs + files in one searchable stream",
```

**Trope**: "One searchable stream" is vague product-speak. "Stream" here is jargon borrowed from activity feeds; it does not accurately describe what Google Drive provides. This is AI-copy filler that sounds technical without being specific.
**Fix**: Replace with a plain description: "Google Drive files and folders" or "Files, docs, and shared drives".

---

### 6. "Files in one searchable stream" (OneDrive)

**File**: `core-integration-card.tsx`
**Line**: 28
**Offending text**:

```typescript
"microsoft-onedrive": "Files in one searchable stream",
```

**Trope**: Same "searchable stream" jargon as violation 5, applied to OneDrive.
**Fix**: "OneDrive files and folders" or "Files and shared documents".

---

## No-violation files

- `connection-wizard/stepper-icon.tsx`: Pure icon logic, no user-facing copy.
- `integration-catalog.tsx`: Configuration and utility functions, no user-facing copy.
- `integration-connection-row.tsx`: Functional labels only ("Syncing...", "Access: Read-only", "Sync failed"). No tropes found.
- `integration-row.tsx`: Status labels and "Coming soon" badge. No tropes found.

---

## Summary

| #   | File                          | Line(s) | Issue                                                                             |
| --- | ----------------------------- | ------- | --------------------------------------------------------------------------------- |
| 1   | core-integration-card.tsx     | 23, 29  | "Email threads + suggested replies" — identical filler copy for Gmail and Outlook |
| 2   | directory-view.tsx            | 92-93   | "Connect your tools" / "Choose what to sync" — generic onboarding header          |
| 3   | directory-view.tsx            | 143     | "Connect your tools" — repeated as empty-state section label                      |
| 4   | integration-connect-modal.tsx | 154     | "Connect your X account to start syncing" — filler empty-state text               |
| 5   | core-integration-card.tsx     | 22      | "Docs + files in one searchable stream" — "stream" jargon                         |
| 6   | core-integration-card.tsx     | 28      | "Files in one searchable stream" — same jargon for OneDrive                       |
