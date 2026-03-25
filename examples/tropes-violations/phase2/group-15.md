# Phase 2: Cross-Cutting Patterns (Group 15)

**Slices analyzed**: 12 (packages/ui/src -- atoms/icons, atoms/stories, foundations/hooks, foundations/layout, foundations/stories, foundations/providers, molecules slices 1-3)

---

## Pattern 1: Zero violations across atoms, foundations, and structural molecules

**Slices affected**: 12 of 12

Every file in this group returned zero findings. This covers:

- ~30 SVG icon components (pure path data, no text)
- 6 foundation hooks (pure logic, no text)
- Layout primitives (Stack, StackItem)
- Design token documentation (Storybook MDX and stories)
- Structural molecule components (accordion, alert, alert-dialog, blankslate, breadcrumbs, card family, collapsible, command, dialog, dropdown-menu, empty-state, enhanced-alert)

The UI package's low-level building blocks are consistently free of authored copy. All user-visible text in these components is either passed in via props (caller-controlled) or consists of standard functional labels ("Close", "Previous", "Next", accessibility aria-labels).

This is the expected outcome for a well-structured Atomic Design system where atoms and molecules are presentation-only and content is injected from above.

---

## Pattern 2: Storybook fixture data is consistently clean

**Slices affected**: atoms/stories, foundations/stories, foundations/layout, molecules slices 1-3 (6 of 12)

Story files across this group use technical, factual placeholder text:

- Developer documentation labels ("Vertical Stack", "Default vertical layout with normal gap (16px)")
- Standard form placeholders ("Enter your name", "Enter your email")
- Generic person data ("John Doe", "john@example.com")
- Accurate component descriptions ("Basic line chart with two data series")

No AI writing tropes leaked into Storybook fixture data at this layer. This is a positive pattern: story authors are not seeding trope-laden copy that downstream component consumers might copy-paste into production.

---

## Pattern 3: JSDoc examples use plain functional copy

**Slices affected**: molecules slices 1-3 (blankslate, empty-state, enhanced-alert, flash, command)

JSDoc `@example` blocks in molecule components model reasonable copy patterns:

- `"No messages yet"` / `"When you receive messages, they'll appear here"` (blankslate)
- `"No messages"` / `"Your inbox is empty. Connect an integration to get started."` (empty-state)
- `"Code Agent Failed"` with `"Retry"`, `"View Logs"` labels (enhanced-alert)
- `"Command Palette"` / `"Search for a command to run..."` (command)

These examples are functional and free of AI tropes. They set a reasonable copy direction for consumers of these components. Worth preserving as-is.

---

## Summary

This group is clean. The atoms, foundations, and structural molecules in `packages/ui/src` contain no AI writing tropes. All user-visible text is either caller-injected or consists of standard UI labels. Storybook stories and JSDoc examples model good copy patterns. No action needed.
