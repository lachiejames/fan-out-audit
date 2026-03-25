# Tropes Audit: packages/ui/src/molecules/EntityBadge

Files reviewed:

- `packages/ui/src/molecules/EntityBadge/EntityBadge.tsx`
- `packages/ui/src/molecules/EntityBadge/EntityBadge.stories.tsx`

---

## EntityBadge.tsx

**JSDoc description (lines 18-24):**

> "EntityBadge - Shows cross-platform identity resolution"
> "Displays name → Slack handle → directory path derived from email."

Internal developer documentation. The term "identity resolution" is technical jargon, not user-visible. No findings.

**Rendered output format (lines 44-49):**

```tsx
<span>({name}</span>
<span>→</span>
<span>{slackHandle}</span>
<span>→</span>
<span>{directoryName})</span>
```

The component renders identity mappings like `(John Doe → @johndoe → john-doe)` directly as user-visible text. This is a functional display of derived identifiers. The format is minimalist and data-driven. No AI trope language. No findings.

---

## EntityBadge.stories.tsx

**Story data (lines 19-37):**

> `email: "sarah.chen@company.com"` / `name: "Sarah Chen"` / `email: "john@example.com"` / `name: "John"` / `email: "alexander.von.humboldt@organization.com"` / `name: "Alexander von Humboldt"`

Generic example data. No findings.

**Story labels (lines 43-57):**

> `"Standard:"` / `"Dots in name:"` / `"Simple:"` / `"Long email:"`

Developer story descriptions. No findings.

**InContext story (lines 62-69):**

> `"Sarah Chen"` / `"2 hours ago"`

Plain data. No findings.

---

## Summary

No AI writing tropes found. EntityBadge is purely a data display component with no marketing or AI-framing copy.
