# Audit: apps/app/src/components — Slice 7 (billing components part 3)

**Files inspected**: 8
**Findings**: 4

## Summary

This slice continues the `tierUpgrades` duplicate pattern from slice 6 (same structure in a different component), and adds two new cast patterns: bracket-notation access on a recharts payload and a `Record<string, typeof Icon>` map with untyped keys.

## Findings

### Finding 1: Duplicate `tierUpgrades` record pattern in `sync-limit-paywall.tsx`

- **File**: `apps/app/src/components/billing/sync-limit-paywall.tsx`
- **Category**: Duplicate type definitions
- **Impact**: high
- **Description**: `sync-limit-paywall.tsx` defines a `tierUpgrades` record with the same shape as the one in `soft-paywall.tsx` (slice 6). This is a structural duplicate — two components independently define the same tier-to-upgrade-descriptor mapping with the same loose `Record<string, ...>` types. If a new tier is added or the upgrade path changes, both files must be updated.
- **Suggestion**: Extract the tier upgrade map to a shared module (e.g. `apps/app/src/components/billing/billing.constants.ts`) and import it in both paywall components. Apply the `Record<SubscriptionTier | 'free', ...>` fix from slice 6 Finding 1 once, in the shared location.
- **Evidence**: Parallel `tierUpgrades` constant in both `soft-paywall.tsx` and `sync-limit-paywall.tsx` with identical structure.

### Finding 2: Recharts payload access via string key cast

- **File**: `apps/app/src/components/billing/usage-chart-content.tsx:101`
- **Category**: Type casts (`as SomeType`) that could be avoided
- **Impact**: medium
- **Description**: `const dateKey = item?.payload?.["date"] as string | undefined` accesses a recharts tooltip payload entry via bracket notation with a cast. Recharts types the payload as `any` internally, so this pattern propagates the unsafety.
- **Suggestion**: Define a typed interface for the chart data points (e.g. `interface UsageChartEntry { date: string; count: number; ... }`) and use it as the generic type parameter for the recharts component and tooltip props. Access `item.payload.date` without a cast once the type is known.
- **Evidence**: `item?.payload?.["date"] as string | undefined` at line 101.

### Finding 3: `TRANSACTION_ICON_MAP` uses `string` keys for known transaction types

- **File**: `apps/app/src/components/billing/transaction-item.tsx:7`
- **Category**: `Record<string, ...>` where stricter types would be safer
- **Impact**: low
- **Description**: `const TRANSACTION_ICON_MAP: Record<string, typeof CalendarCheck>` maps transaction type strings to Lucide icon components. The set of valid transaction types is a closed set determined by the API.
- **Suggestion**: Check `@slopweaver/contracts` for a `TransactionType` or similar type. If present, use `Record<TransactionType, typeof CalendarCheck>`. If not, define a local `type TransactionType = 'subscription' | 'refund' | ...` union. This ensures all transaction types have an icon and prevents silent undefined lookups.
- **Evidence**: `const TRANSACTION_ICON_MAP: Record<string, typeof CalendarCheck>` at line 7.

### Finding 4: Billing chart data interface not shared between chart and tooltip components

- **File**: `apps/app/src/components/billing/usage-chart-content.tsx`
- **Category**: Duplicate type definitions
- **Impact**: low
- **Description**: The chart data shape (date, usage count, etc.) is implicitly typed through recharts generics in the chart component and separately assumed in the tooltip render function. There is no single shared typed interface for the chart entry, leading to the cast in Finding 2 and potential drift.
- **Suggestion**: Define `interface UsageChartEntry { ... }` once at the top of the file or in a collocated types file, and use it as the type parameter for both the chart and tooltip components.
- **Evidence**: Inconsistent typing between chart data array and tooltip payload access pattern.
