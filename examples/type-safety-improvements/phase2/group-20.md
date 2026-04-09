# Cross-Cutting Patterns (Group 20)

**Slices analyzed**: packages-ui-src-atoms-4, packages-ui-src-atoms-5, packages-ui-src-atoms-6, packages-ui-src-atoms-7, packages-ui-src-atoms-8, packages-ui-src-atoms-9, packages-ui-src-foundations-1, packages-ui-src-foundations-2, packages-ui-src-foundations-3, packages-ui-src-foundations-4, packages-ui-src-index.ts, packages-ui-src-molecules-1

## Pattern 1: Polymorphic `as` prop cast duplicated across 4+ components

- **Seen in**: packages-ui-src-atoms-4 (scroll-area forwardRef, related pattern), packages-ui-src-atoms-8 (flash.tsx), packages-ui-src-foundations-4 (stack.tsx/StackItem)
- **Category**: type-cast
- **Combined impact**: Medium -- the same `const Component = as as React.ElementType` and `ref as React.Ref<HTMLElement>` cast pair is independently written in Box, Text, Flash, Stack, and StackItem. Five components carry two cast sites each (10 total casts) with no shared abstraction.
- **What's happening**: Every polymorphic component that accepts an `as` prop to render as a different HTML element must cast the element type and the ref. Each component re-implements these casts independently, creating maintenance burden and inconsistent documentation (some have explanatory comments, some do not).
- **Suggestion**: Extract a shared `createPolymorphicComponent` factory or a `PolymorphicBox` base component that centralises both casts in one place. All five components would delegate to it, reducing the 10 cast sites to 2.
- **Estimated scope**: 5 components (Box, Text, Flash, Stack, StackItem), ~10 cast sites

## Pattern 2: Avoidable type casts from incomplete narrowing

- **Seen in**: packages-ui-src-foundations-1 (use-responsive-value.tsx), packages-ui-src-foundations-2 (platform-styles.ts), packages-ui-src-foundations-3 (react-children-helpers.ts), packages-ui-src-molecules-1 (dialog.tsx)
- **Category**: type-cast
- **Combined impact**: Medium -- at least 12 individual cast sites across 4 files follow the same pattern: a runtime check proves a value's type, but TypeScript cannot narrow it, so a manual `as` cast bridges the gap.
- **What's happening**: Multiple files use `as` casts immediately after a runtime check that already establishes the correct type. Examples: `typeof x === "function"` followed by `x as React.ComponentType`; `in` check followed by `obj[key as keyof typeof obj]!`; null check followed by `as HTMLElement`; property access on a discriminated union followed by `as TNarrow`. In each case, a custom type guard or `instanceof` check would let TypeScript narrow automatically.
- **Suggestion**: For each pattern, replace the `as` cast with the appropriate narrowing mechanism: `instanceof` guards for DOM types, custom type predicates (`value is ComponentType`) for function checks, `"key" in obj` narrowing for discriminated unions, and `Object.hasOwn` with typed keys for record lookups.
- **Estimated scope**: 4 files, ~12 cast sites

## Pattern 3: Duplicate type definitions that can diverge silently

- **Seen in**: packages-ui-src-foundations-1 (MediaQueryFeatures in match-media.tsx + use-media.tsx), packages-ui-src-foundations-2 (ColorPalette interface vs lightColors/darkColors objects)
- **Category**: duplicate-type
- **Combined impact**: Medium -- two independent instances of the same anti-pattern: a type is manually declared in multiple locations (or manually declared to mirror a const object) rather than derived from a single source of truth. Changes to one copy require remembering to update the other.
- **What's happening**: `MediaQueryFeatures` is identically declared in two related hook files. `ColorPalette` is a 50+ field interface that mirrors the shape of `lightColors` and `darkColors` const objects. In both cases, the duplicate can silently diverge from the source of truth.
- **Suggestion**: For `MediaQueryFeatures`, extract to a shared `media-query-types.ts` and import in both files. For `ColorPalette`, replace the manual interface with `type ColorPalette = typeof lightColors` and add `satisfies ColorPalette` to `darkColors`.
- **Estimated scope**: 3 files, 2 type definitions

## Pattern 4: `Record<string, unknown>` used as React props escape hatch

- **Seen in**: packages-ui-src-foundations-3 (react-children-helpers.ts -- 3 sites), packages-ui-src-foundations-3 (iconOrElement cast)
- **Category**: record-weakening
- **Combined impact**: Low -- all 3 sites in react-children-helpers.ts cast `element.props` to `Record<string, unknown>` because React types `ReactElement.props` as `{}`. This is a React API limitation, not a codebase design choice, but the pattern appears 3 times in one file with no documentation.
- **What's happening**: React's `ReactElement.props` type is `{}` (opaque). To access child props by key (e.g., `.className`, `.disabled`), the code must cast to `Record<string, unknown>`. The cast is correct but undocumented at all 3 sites.
- **Suggestion**: Add a shared helper like `function getElementProps(element: React.ReactElement): Record<string, unknown>` with a single documented cast, and call it from all 3 sites. Alternatively, add a brief comment at each site explaining the React limitation.
- **Estimated scope**: 1 file, 3 cast sites

## Pattern 5: Majority of UI atoms are type-safe (positive signal)

- **Seen in**: packages-ui-src-atoms-5, packages-ui-src-atoms-6, packages-ui-src-atoms-7, packages-ui-src-atoms-9, packages-ui-src-index.ts
- **Category**: N/A (positive pattern)
- **Combined impact**: N/A -- 5 of 12 slices reported zero findings. The standard `React.ComponentProps<typeof Primitive>` pattern used by Radix wrappers is consistently type-safe across ~30 atom components.
- **What's happening**: The codebase has a well-established pattern for wrapping Radix UI primitives that avoids casts, `any`, and duplicate types. The type-safety issues are concentrated in polymorphic components (Pattern 1), utility functions (Patterns 2/4), and manual type declarations (Pattern 3) rather than in the component library's bread-and-butter atoms.
- **Suggestion**: No action needed. This confirms that remediation effort should focus on the polymorphic component abstraction and foundation utilities, not the atom layer.
- **Estimated scope**: N/A
