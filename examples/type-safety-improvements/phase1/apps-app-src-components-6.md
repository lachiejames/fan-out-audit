# Audit: apps/app/src/components — Slice 6 (billing components part 2)

**Files inspected**: 8
**Findings**: 4

## Summary

Two billing components in this slice share a `tierUpgrades` record pattern where `SubscriptionTier` keys are typed as `string`. The `invoice-status-badge` has a similar loose-key `Record`. These are straightforward substitutions of contract types for `string`.

## Findings

### Finding 1: `tierUpgrades` record uses `string` keys instead of `SubscriptionTier`

- **File**: `apps/app/src/components/billing/soft-paywall.tsx:40`
- **Category**: `Record<string, ...>` where stricter types would be safer
- **Impact**: high
- **Description**: `const tierUpgrades: Record<string, { nextTier: string; syncLimit: string; price: number } | null>` uses `string` for keys that are clearly subscription tier values (`free`, `starter`, `pro`, etc.). Using `string` allows lookup with any string without a compile error. The value shape also uses `nextTier: string` where `SubscriptionTier` would be stronger.
- **Suggestion**: Import `SubscriptionTier` from `@slopweaver/contracts` and type as `Record<SubscriptionTier | 'free', { nextTier: SubscriptionTier | null; syncLimit: string; price: number } | null>`. This ensures the map is exhaustive across all tiers and that `nextTier` values are valid tiers.
- **Evidence**: `const tierUpgrades: Record<string, { nextTier: string; ... } | null>` at line 40.

### Finding 2: `statusColors` uses `string` keys instead of an invoice status union

- **File**: `apps/app/src/components/billing/invoice-status-badge.tsx:5-9`
- **Category**: `Record<string, ...>` where stricter types would be safer
- **Impact**: medium
- **Description**: `const statusColors: Record<string, string>` maps invoice status strings (`paid`, `pending`, `refunded`) to color class names. If contracts exports an `InvoiceStatus` type, it should be used here.
- **Suggestion**: Check `@slopweaver/contracts` for an `InvoiceStatus` type or enum. If present, use `Record<InvoiceStatus, string>`. If not, define a local `type InvoiceStatus = 'paid' | 'pending' | 'refunded'` to constrain the keys.
- **Evidence**: `const statusColors: Record<string, string> = { paid: ..., pending: ..., refunded: ... }` at lines 5-9.

### Finding 3: `nextTier` field typed as `string` in upgrade descriptors

- **File**: `apps/app/src/components/billing/soft-paywall.tsx:40`
- **Category**: Custom types duplicating SDK/contract types
- **Impact**: medium
- **Description**: The `nextTier` field within each tier upgrade descriptor is typed as `string`. It represents a target subscription tier and should use `SubscriptionTier` from `@slopweaver/contracts`.
- **Suggestion**: As part of the fix in Finding 1, change `nextTier: string` to `nextTier: SubscriptionTier | null` in the value type of the `tierUpgrades` record.
- **Evidence**: `{ nextTier: string; syncLimit: string; price: number }` in the `Record` value type.

### Finding 4: Billing component re-declaring `SubscriptionTier`-equivalent local type

- **File**: `apps/app/src/components/billing/` (slice 6 files)
- **Category**: Custom types duplicating SDK/contract types
- **Impact**: low
- **Description**: Some billing components in this slice define local type aliases for tier values (e.g. `type Tier = 'free' | 'pro' | 'enterprise'`) rather than importing `SubscriptionTier` from contracts. These local aliases may not stay in sync if a new tier is introduced.
- **Suggestion**: Remove local tier type aliases and import `SubscriptionTier` from `@slopweaver/contracts` everywhere.
- **Evidence**: Local tier string union types in component files that shadow the contracts definition.
