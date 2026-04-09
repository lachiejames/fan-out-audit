# Audit: packages/ui/src/organisms (slice 1)

**Files inspected**

- `packages/ui/src/organisms/AiPresenceLogo/AiPresenceLogo.tsx`
- `packages/ui/src/organisms/Calendar/MeetingCard.tsx`
- `packages/ui/src/organisms/Calendar/types.ts`
- `packages/ui/src/organisms/chat/code-block-context.tsx`
- `packages/ui/src/organisms/chat/code-block-copy-button.tsx`
- `packages/ui/src/organisms/chat/code-block.tsx`
- `packages/ui/src/organisms/chat/controls.tsx`

**Findings**: 2 findings

---

## Summary

`AiPresenceLogo.tsx` uses `null` as the default value for all optional props (rather than optional `?` props), which is a non-standard pattern that requires consumers to pass `null` explicitly instead of omitting the prop. The chat files are clean.

---

## Findings

### Finding 1: All props in `AiPresenceLogoProps` are `T | null` instead of optional (`?`)

- **File**: `packages/ui/src/organisms/AiPresenceLogo/AiPresenceLogo.tsx` (lines 18–29)
- **Category**: Missing explicit prop types / inferred-any leaking
- **Impact**: Medium — consumers must pass `null` explicitly for every optional prop rather than omitting them; this is inconsistent with the rest of the design system and adds unnecessary friction at call sites
- **Description**: Every prop in `AiPresenceLogoProps` is typed as `T | null` (e.g., `state: AiPresenceState | null`, `onClick: (() => void) | null`). The component then uses `?? defaultValue` to handle the null. The standard React pattern for optional props is `state?: AiPresenceState` (i.e., `T | undefined`) with a default parameter value, not `T | null` with nullish coalescing.
- **Suggestion**: Change all `T | null` props to optional `T?` (i.e., `T | undefined`) and use default parameter values in the destructuring. This aligns with the rest of the design system and allows callers to simply omit props they don't need.
- **Evidence**:
  ```typescript
  export interface AiPresenceLogoProps {
    state: AiPresenceState | null;
    size: AiPresenceSize | null;
    onClick: (() => void) | null;
    // ...
  }
  ```

---

### Finding 2: `Meeting.color` typed as `string` instead of a Tailwind gradient class union

- **File**: `packages/ui/src/organisms/Calendar/types.ts` (line 24)
- **Category**: Missing explicit prop types / inferred-any leaking
- **Impact**: Low — the string type is intentional (Tailwind classes are arbitrary strings), but the JSDoc clarifies this is specifically for gradient classes
- **Description**: `Meeting.color: string` is documented as `"Tailwind gradient classes e.g. 'from-blue-600 to-blue-700'"`. While a full union of all Tailwind classes is impractical, the component would benefit from a type alias like `type TailwindGradient = string` with a JSDoc comment to signal intent and aid documentation tools.
- **Suggestion**: Add a type alias `type TailwindClassName = string` with documentation, or at minimum strengthen the JSDoc with an `@example` showing valid values. This is low priority but improves discoverability.
- **Evidence**: `/** Tailwind gradient classes e.g. "from-blue-600 to-blue-700" */ color: string;`
