# Audit: packages/ui/src/atoms (slice 2)

**Files inspected**

- `packages/ui/src/atoms/icons/platform-icon-map.ts`
- `packages/ui/src/atoms/context-chip.tsx`

**Findings**: 4 findings

---

## Summary

Both files deal with platform identifiers but use `string` where a narrower union type would eliminate runtime casts and improve discoverability. `platform-icon-map.ts` casts `PlatformIconMap` to a loose record to support cross-map fallback lookups. `context-chip.tsx` uses `Record<string, ...>` maps and a `string` prop for platform where `PlatformId` (or a local union) would be more precise.

---

## Findings

### Finding 1: `PlatformIconMap` cast to `Record<string, ...>` in lookup function

- **File**: `packages/ui/src/atoms/icons/platform-icon-map.ts` (lines 111–113)
- **Category**: Type cast (`as SomeType`) avoidable with better typing
- **Impact**: Medium — the cast discards the precise `Record<PlatformId, ...>` type, allowing any string through without a compile error
- **Description**: `getPlatformIcon` casts `PlatformIconMap as Record<string, ((props: PlatformIconProps) => React.JSX.Element) | undefined>` to perform a runtime fallback lookup across multiple maps. This is necessary because `platform` is typed as `string`, not `PlatformId`. If `platform` were narrowed before the lookup (using the existing `isPlatformId` guard), the cast would be unnecessary for the primary path.
- **Suggestion**: Accept `platform: string` externally but internally call `isPlatformId` to narrow, then look up in the strongly-typed map without casting. Reserve the cast only for the `ExtraIconMap` fallback path, which is intentionally loose.
- **Evidence**:
  ```typescript
  return (
    (PlatformIconMap as Record<string, ((props: PlatformIconProps) => React.JSX.Element) | undefined>)[platform] ??
    NonPlatformIconMap[platform as NonPlatformIconId] ??
    ExtraIconMap[platform]
  );
  ```

---

### Finding 2: `ExtraIconMap: Record<string, ...>` — missing `ExtraIconId` union

- **File**: `packages/ui/src/atoms/icons/platform-icon-map.ts` (line 82)
- **Category**: `Record<string, ...>` where stricter keys would be safer
- **Impact**: Low — the map is defined locally and its keys are finite
- **Description**: `ExtraIconMap` is typed as `Record<string, (props: PlatformIconProps) => React.JSX.Element>` with a comment saying it's intentionally loose. However, the map only contains a known finite set of keys (`"ai"`, `"knowledge"`, etc.). Defining an `ExtraIconId` union and using `Record<ExtraIconId, ...>` would prevent typos and allow exhaustive checks.
- **Suggestion**: Extract `type ExtraIconId = "ai" | "knowledge" | ...` and type the map as `Record<ExtraIconId, ...>`.
- **Evidence**: `const ExtraIconMap: Record<string, (props: PlatformIconProps) => React.JSX.Element> = { ... }`

---

### Finding 3: `PLATFORM_STYLES` and `PLATFORM_ICONS` keyed by `string` in ContextChip

- **File**: `packages/ui/src/atoms/context-chip.tsx` (lines 33, 78)
- **Category**: `Record<string, ...>` where stricter union keys would be safer
- **Impact**: Medium — typos in platform keys fail silently at runtime (returns `undefined`)
- **Description**: Both `PLATFORM_STYLES: Record<string, string>` and `PLATFORM_ICONS: Record<string, React.ReactNode>` use broad string keys even though their keys are a finite set matching `PlatformId`. A lookup with an unknown platform string currently returns `undefined` silently.
- **Suggestion**: Import `PlatformId` from `@slopweaver/contracts` and type both maps as `Partial<Record<PlatformId, ...>>` (partial because not every platform necessarily has an entry). This converts silent `undefined` returns into typed `string | undefined` / `ReactNode | undefined`.
- **Evidence**: `const PLATFORM_STYLES: Record<string, string> = { "google-gmail": "...", slack: "...", ... }`

---

### Finding 4: `platform: string` prop in ContextChipProps

- **File**: `packages/ui/src/atoms/context-chip.tsx` (ContextChipProps interface)
- **Category**: Missing explicit prop types / inferred-any leaking
- **Impact**: Low — callers can pass any string without a compile error
- **Description**: `platform` is typed as `string` rather than `PlatformId | string` (a branded union). While a fully strict `PlatformId` would break callers that pass arbitrary strings, using `PlatformId | (string & {})` preserves autocomplete for known values while still accepting arbitrary strings.
- **Suggestion**: Type `platform` as `PlatformId | (string & {})` to preserve known-value autocomplete without breaking callers.
- **Evidence**: `platform: string;` in `ContextChipProps`
