# Audit: packages/ui/src/molecules (slice 2)

**Files inspected**

- `packages/ui/src/molecules/chart.tsx`
- `packages/ui/src/molecules/collapsible.tsx`
- `packages/ui/src/molecules/tabs.tsx`

**Findings**: 3 findings

---

## Summary

`chart.tsx` has the most type-safety issues in the molecules layer: loose index signatures on payload interfaces and multiple string casts in the config lookup function. `collapsible.tsx` and `tabs.tsx` are clean.

---

## Findings

### Finding 1: `[key: string]: unknown` index signature on `TooltipPayloadItem` and `LegendPayloadItem`

- **File**: `packages/ui/src/molecules/chart.tsx` (lines 163, 367)
- **Category**: `Record<string, ...>` where stricter keys would be safer
- **Impact**: Medium — any access to these types loses type information; all properties require casts downstream
- **Description**: Both `TooltipPayloadItem` and `LegendPayloadItem` interfaces include `[key: string]: unknown` index signatures alongside named fields. The index signature was added to accommodate arbitrary recharts payload structure, but it means every named property (`name`, `value`, `color`) is also typed as `unknown` through the index signature, forcing downstream casts.
- **Suggestion**: Remove the index signature and instead define a separate `unknownPayload: Record<string, unknown>` field for extra data. This preserves type safety on the known fields while still allowing arbitrary extra data to be stored.
- **Evidence**:
  ```typescript
  interface TooltipPayloadItem {
    name?: string;
    value?: number | string;
    color?: string;
    [key: string]: unknown; // Remove this
  }
  ```

---

### Finding 2: Multiple `as string` casts in `getPayloadConfigFromPayload`

- **File**: `packages/ui/src/molecules/chart.tsx` (lines 455–462)
- **Category**: Type cast (`as SomeType`)
- **Impact**: Medium — if the payload structure changes, these casts silently produce `undefined` cast to `string`
- **Description**: `getPayloadConfigFromPayload` accesses `payloadObj["payload"] as Record<string, unknown>` and then `payloadObj[key] as string` and `payloadPayload[key] as string`. These casts are forced by the `[key: string]: unknown` index signature on `TooltipPayloadItem`. Fixing Finding 1 would eliminate the need for these casts.
- **Suggestion**: Resolves automatically if Finding 1 is addressed. Until then, add runtime `typeof` guards before the `as string` casts.
- **Evidence**:
  ```typescript
  const payloadObj = payload as Record<string, unknown>;
  payloadObj[key] as string;
  payloadPayload[key] as string;
  ```

---

### Finding 3: `payload as Record<string, unknown>` cast in config lookup

- **File**: `packages/ui/src/molecules/chart.tsx` (line 450)
- **Category**: Type cast (`as SomeType`)
- **Impact**: Low — same root cause as Finding 2
- **Description**: The `payload` parameter in `getPayloadConfigFromPayload` is typed as `TooltipPayloadItem`, yet it is immediately cast to `Record<string, unknown>` for key access. This is a sign that `TooltipPayloadItem`'s type is insufficient.
- **Suggestion**: Resolves if `TooltipPayloadItem` is properly typed without the index signature.
- **Evidence**: `const payloadObj = payload as Record<string, unknown>;`
