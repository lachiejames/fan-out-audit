# Audit: apps-app-src-components-12

**Files inspected**:

- `apps/app/src/components/inbox/smart-snooze-picker.tsx`
- `apps/app/src/components/inbox/suggestion-chips.tsx`
- `apps/app/src/components/inbox/suggestion-explanation.tsx`
- `apps/app/src/components/inbox/suggestion-reasoning-sheet.tsx`
- `apps/app/src/components/inbox/suggestion-step-card.tsx`
- `apps/app/src/components/integrations/connection-wizard/connection-wizard-config.ts`
- `apps/app/src/components/integrations/connection-wizard/connection-wizard-context.ts`
- `apps/app/src/components/integrations/connection-wizard/connection-wizard-reducer.ts`

**Findings**: 3

---

## Summary

Three type-safety gaps in this slice. Two are `Record<string, T>` icon maps that should use narrower literal union keys. One is a wizard step array typed as `readonly { id: string; label: string }[]` that could be inferred from a `typeof` of the constant. The smart-snooze `SnoozeOption` local type is an intentional `exactOptionalPropertyTypes` workaround and is acceptable.

---

## Findings

### Finding 1: `Record<string, React.ReactNode>` for icon map in suggestion-chips.tsx

- **File**: `apps/app/src/components/inbox/suggestion-chips.tsx`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `iconMap` at line 38 is typed `Record<string, React.ReactNode>`. The keys are the `icon` field values from `SuggestionChip`, which at line 29 is typed as `string`. Nothing prevents an unknown icon key from being passed in; the map will return `undefined` silently.
- **Suggestion**: Define a `const ICON_MAP = { email: ..., calendar: ..., ... } as const` and derive `type IconKey = keyof typeof ICON_MAP`. Type `SuggestionChip.icon` as `IconKey` and use `Record<IconKey, React.ReactNode>` for the map. This makes invalid icon names a compile error.
- **Evidence**:
  ```typescript
  // line 29
  icon: string
  // line 38
  const iconMap: Record<string, React.ReactNode> = { ... }
  ```

---

### Finding 2: `Record<string, React.ReactNode>` for step icon map in suggestion-step-card.tsx

- **File**: `apps/app/src/components/inbox/suggestion-step-card.tsx`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `stepIconMap` at line 19 is typed `Record<string, React.ReactNode>`. The same issue as Finding 1: unknown step type names silently return `undefined`.
- **Suggestion**: Extract step type names as a literal union or `const` object and narrow the map key type.
- **Evidence**:
  ```typescript
  // line 19
  const stepIconMap: Record<string, React.ReactNode> = { ... }
  ```

---

### Finding 3: Wizard steps typed as `readonly { id: string; label: string }[]` instead of `typeof CONNECT_WIZARD_STEPS`

- **File**: `apps/app/src/components/integrations/connection-wizard/connection-wizard-context.ts`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `wizardSteps: readonly { id: string; label: string }[]` at line 31 is a loose structural type. If `CONNECT_WIZARD_STEPS` is a `const` array elsewhere, the context type should be `typeof CONNECT_WIZARD_STEPS` so that narrowing on step `id` can be exhaustive.
- **Suggestion**: Import the constant and use `typeof CONNECT_WIZARD_STEPS` as the type for `wizardSteps` in the context interface.
- **Evidence**:
  ```typescript
  // line 31
  wizardSteps: readonly { id: string; label: string }[];
  ```
