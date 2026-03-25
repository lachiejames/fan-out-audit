# Tropes Audit: packages/ui/src/foundations/stories (Slice 175)

Files audited:

- `packages/ui/src/foundations/stories/design-tokens.mdx`
- `packages/ui/src/foundations/stories/design-tokens/colors.stories.tsx`
- `packages/ui/src/foundations/stories/design-tokens/typography.stories.tsx`
- `packages/ui/src/foundations/stories/design-tokens/spacing.stories.tsx`
- `packages/ui/src/foundations/stories/design-tokens/shadows.stories.tsx`
- `packages/ui/src/foundations/stories/design-tokens/other-tokens.stories.tsx`
- `packages/ui/src/foundations/stories/design-tokens/animations.stories.tsx`

Note: All of these are internal Storybook documentation files visible only to developers.

---

## design-tokens.mdx

This is developer documentation for the design system. It references historical internal naming conventions ("v0 2025", "Premium brand colors with vibrant, energetic v0 identity"). These are internal dev notes, not user-facing.

One phrase to note: **"Premium Animations (2025)"** as a section heading -- this is developer documentation describing animation token categories. Not user-facing copy, but the word "Premium" is mildly marketing-inflected even in internal docs. Low severity, documentation-only.

No actionable violations -- this is a Storybook MDX doc page, not product UI.

---

## colors.stories.tsx

No findings. Color documentation with technical descriptions. Story titles and descriptions are accurate and factual: "Complete color palette for SlopWeaver. Copy CSS variable names for use in components." Developer-facing.

The story descriptions use "v0 brand identity colors" and "v0 palette" -- these are internal codenames referencing an earlier design iteration, not marketing copy.

---

## typography.stories.tsx

No findings. Typography documentation with technical specimen text ("The quick brown fox jumps over the lazy dog" -- the standard typographic pangram). All descriptions are functional and factual. Story description: "Complete typography scale for SlopWeaver. Copy CSS variable names for consistent text styling." Developer-facing.

---

## spacing.stories.tsx

No findings. Spacing documentation with technical descriptions. All content is factual. Story description: "Complete spacing scale for SlopWeaver. Copy CSS variable names for consistent spacing." Developer-facing.

---

## shadows.stories.tsx

No findings. Shadow documentation. All descriptions are functional and factual. Story description: "Complete shadow scale for SlopWeaver. Copy CSS variable names for consistent depth and elevation." Developer-facing.

---

## other-tokens.stories.tsx

No findings. Documentation for border radius, z-index, breakpoints, and container sizes. All descriptions are functional and factual. Developer-facing.

---

## animations.stories.tsx

No findings. Animation token documentation. Descriptions like "2s ease-in-out infinite - Pulsing glow effect", "3s ease-in-out infinite - Floating motion", "0.3s ease-out - Slide from right" are technical and factual. The button label "Click to preview" is a functional UI label within a Storybook demo, not product copy. Developer-facing.

---

## Summary

**Total violations: 0**

All foundation story files are internal Storybook developer documentation. They contain technical descriptions and standard developer placeholder patterns. No AI writing tropes present in user-facing contexts.

One minor internal note: the MDX file uses "Premium Animations (2025)" as a section heading in developer docs. This is not actionable -- it's an internal doc artifact, not product copy.
