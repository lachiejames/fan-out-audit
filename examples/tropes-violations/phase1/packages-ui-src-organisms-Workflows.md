# Slice 194: packages/ui/src/organisms/Workflows

Files audited:

- `packages/ui/src/organisms/Workflows/workflow-node.tsx`
- `packages/ui/src/organisms/Workflows/WorkflowNode.stories.tsx`

---

## workflow-node.tsx

**User-facing strings:**

- `"Edit"` and `"Delete"` — hover action buttons on nodes. Functional labels. No violations.
- `node.type` rendered as uppercase tracking text (e.g. "trigger", "action", "condition", "ai"). These are data-driven from the node type enum, not authored copy.

---

## WorkflowNode.stories.tsx

Story-only file. Story descriptions ("Trigger nodes start workflows. Cyan gradient with mail icon.", "AI nodes use machine learning. Pink gradient with sparkles icon.") are internal Storybook docs, not user-facing product copy.

The description "AI nodes use machine learning" is technically inside a Storybook `docs.description.story` field visible only to developers in Storybook. Not a user-facing violation.

---

## Findings

No AI writing tropes found in this slice.
