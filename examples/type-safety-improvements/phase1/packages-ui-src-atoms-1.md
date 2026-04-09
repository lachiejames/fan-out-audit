# Audit: packages/ui/src/atoms (slice 1)

**Files inspected**

- `packages/ui/src/atoms/box.tsx`
- `packages/ui/src/atoms/text.tsx`
- `packages/ui/src/atoms/icon.tsx`

**Findings**: 3 findings

---

## Summary

The three polymorphic atoms (`Box`, `Text`, `Icon`) all have minor type-safety gaps rooted in the same cause: TypeScript cannot express fully generic polymorphic `as`-prop types without escape hatches. Two of the three files use acknowledged casts. `Icon` is missing an explicit return type.

---

## Findings

### Finding 1: Unsafe `as React.ElementType` cast in polymorphic Box

- **File**: `packages/ui/src/atoms/box.tsx` (line 86)
- **Category**: Type cast (`as SomeType`) avoidable with better typing
- **Impact**: Low — cast is structurally necessary; the comment acknowledges the limitation
- **Description**: The polymorphic `as` prop must be cast `as React.ElementType` before passing to JSX because TypeScript cannot narrow the generic `ElementType` at the render site. A second cast `ref as React.Ref<HTMLElement>` appears on line 92 for the same reason.
- **Suggestion**: No ergonomic workaround exists without a separate overload-based approach. Document with `// cast required: TypeScript cannot infer ref type from polymorphic 'as' prop` so future readers understand this is deliberate.
- **Evidence**: `const Component = as as React.ElementType;` and `ref={ref as React.Ref<HTMLElement>}`

---

### Finding 2: `ref as never` escape hatch in Text

- **File**: `packages/ui/src/atoms/text.tsx` (line 48)
- **Category**: Type cast (`as SomeType`)
- **Impact**: Low — `never` cast is a stronger escape hatch than needed and silences the compiler completely for this prop
- **Description**: `ref={ref as never}` suppresses the polymorphic ref type error but `as never` is the most unsafe cast available. The existing comment notes this is intentional but doesn't explain why `never` was preferred over a narrower target like `React.Ref<HTMLElement>`.
- **Suggestion**: Replace `ref as never` with `ref as React.Ref<HTMLElement>` to retain at least the structural expectation of a ref object.
- **Evidence**: `ref={ref as never}`

---

### Finding 3: Missing explicit return type on `Icon` function

- **File**: `packages/ui/src/atoms/icon.tsx` (line 42)
- **Category**: Missing explicit prop types / inferred-any leaking
- **Impact**: Low — TypeScript infers `JSX.Element` correctly, but the project rule requires explicit return types on exported functions
- **Description**: The `Icon` function export has no `: React.JSX.Element` return type annotation. All other exported component functions in this package declare explicit return types.
- **Suggestion**: Add `: React.JSX.Element` to the `Icon` function signature.
- **Evidence**: `export function Icon({ name, size, className }: IconProps) {`
