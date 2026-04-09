# Audit: packages/ui/src/foundations (slice 2)

**Files inspected**

- `packages/ui/src/foundations/platform-styles.ts`
- `packages/ui/src/foundations/theme.ts`
- `packages/ui/src/foundations/providers.tsx`

**Findings**: 3 findings

---

## Summary

`platform-styles.ts` has three separate cast sites, all arising from the mismatch between `PlatformId` and the broader `CitationPlatform` type. `theme.ts` manually re-declares a `ColorPalette` interface that duplicates the inferred type of two `const` objects. `providers.tsx` is clean.

---

## Findings

### Finding 1: `PlatformIconMap` cast to `Partial<Record<CitationPlatform, ...>>` in platform-styles.ts

- **File**: `packages/ui/src/foundations/platform-styles.ts` (lines 18–19)
- **Category**: Type cast (`as SomeType`) avoidable with better typing
- **Impact**: Medium — the cast discards the compile-time guarantee that `PlatformIconMap` only contains valid `PlatformId` keys
- **Description**: `PlatformIconMap` is `Record<PlatformId, ...>` but `CitationPlatform` includes non-`PlatformId` values like `"knowledge"`. The cast is needed because `CitationPlatform` is broader than `PlatformId`. The fix is to use `as Partial<Record<PlatformId, ...>>` and handle the `CitationPlatform`-only values separately.
- **Suggestion**: Separate the lookup: if `isPlatformId(platform)`, use `PlatformIconMap[platform]`; otherwise return a fallback. Eliminates the cast entirely.
- **Evidence**: `PlatformIconMap as Partial<Record<CitationPlatform, ComponentType<{className?: string}>>>`

---

### Finding 2: Non-null assertion after `in` check for `LABEL_OVERRIDES`

- **File**: `packages/ui/src/foundations/platform-styles.ts` (line 36)
- **Category**: Type cast (`as SomeType`)
- **Impact**: Low — the `in` check directly precedes the assertion so it cannot be `undefined`, but TypeScript doesn't narrow the index access result
- **Description**: `LABEL_OVERRIDES[platform as keyof typeof LABEL_OVERRIDES]!` combines a cast and a non-null assertion. The `in` check on the previous line guarantees the value exists, but TypeScript doesn't propagate that narrowing to the subsequent index access.
- **Suggestion**: Use a local variable: `const key = platform as keyof typeof LABEL_OVERRIDES; return LABEL_OVERRIDES[key] ?? undefined` without the `!`. Or use `Object.hasOwn` followed by a typed cast only on the key.
- **Evidence**: `LABEL_OVERRIDES[platform as keyof typeof LABEL_OVERRIDES]!`

---

### Finding 3: Manually declared `ColorPalette` interface duplicates `typeof lightColors`

- **File**: `packages/ui/src/foundations/theme.ts` (lines 140–191)
- **Category**: Duplicate type definitions across components
- **Impact**: Medium — adding a color to `lightColors` or `darkColors` requires a matching update to `ColorPalette`; they can silently diverge
- **Description**: `ColorPalette` interface lists ~50 color keys that are already present in the `lightColors` and `darkColors` const objects. Any type mismatch between the interface and the objects will only be caught if a `satisfies` constraint is applied.
- **Suggestion**: Replace the explicit `ColorPalette` interface declaration with `type ColorPalette = typeof lightColors` and add `satisfies ColorPalette` to `darkColors` to ensure both objects share the same keys. This makes the type derived from the source of truth rather than duplicating it.
- **Evidence**: `interface ColorPalette { textPrimary: string; textSecondary: string; ... }` (50+ manually listed fields)
