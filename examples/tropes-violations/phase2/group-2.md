# Cross-Cutting Patterns: Group 2

**Slices analyzed** (12):
digest, feedback-loops, insights, knowledge-sources, llm-calls, media-files, memory, navigation, notifications, personas, prediction, priority-scoring

---

## Pattern 1: Overwhelmingly clean backend infrastructure (10 of 12 slices)

**Slices**: digest, feedback-loops, insights, knowledge-sources, llm-calls, media-files, memory, navigation, personas, priority-scoring

Ten of twelve slices had zero findings. Every one of these slices contained only error type definitions, factory functions, and HTTP error mappers. No user-facing text exists in these files. This is a strong signal that the `application/*/errors/` and `application/*/utils/error-mapper.ts` layer is structurally trope-free because it contains no prose.

**Implication**: Future audits of `application/` can safely skip `errors/` and `utils/error-mapper.ts` files unless they contain user-visible notification text or UI copy. The two slices with findings (notifications, prediction) are exceptions specifically because they contain user-facing strings.

---

## Pattern 2: User-facing strings in unexpected backend locations (2 of 12 slices)

**Slices**: notifications, prediction

The only findings came from files that embed user-facing copy inside backend service or error code, not in a dedicated copy/i18n layer:

| Slice         | File                                | User-facing string                                                                                 |
| ------------- | ----------------------------------- | -------------------------------------------------------------------------------------------------- |
| notifications | `notification-scheduler.service.ts` | `"Your snoozed task ... is ready for attention."`                                                  |
| prediction    | `prediction.errors.ts`              | `"Insufficient actions for this action..."`, `"This feature is not available during the trial..."` |

Both findings share the same root cause: product copy is co-located with backend logic rather than centralized. This makes it harder to audit, harder to keep consistent, and easier for template-assembled strings to slip through without a prose review.

**Recommendation**: Consider extracting user-facing notification bodies and user-visible error messages into a dedicated copy constants file (or at minimum, flag these locations for copy review during PRs).

---

## Pattern 3: Template-assembled strings that read like concatenation artifacts (2 of 12 slices)

**Slices**: notifications, prediction

Both slices that had findings share a specific sub-pattern: strings that were clearly assembled from variable interpolation templates and never read back as complete sentences.

- **prediction**: `"Insufficient actions for this action. Required: ${requiredActions} actions. Current balance: ${currentBalance} actions."` -- "actions" appears three times serving two different meanings (the currency and the operation).
- **notifications**: `"Your snoozed task ... is ready for attention."` -- "ready for attention" is padding that adds no information beyond what the notification title already conveys.

**Recommendation**: Any template string that interpolates variables into a user-facing sentence should be read aloud as a complete sentence during review. The awkwardness in both cases would have been caught immediately by reading the final output.

---

## Pattern 4: Generic feature references in upgrade prompts (1 slice, but systemic risk)

**Slices**: prediction

The trial-locked error message says `"This feature is not available during the trial"` without naming the feature. While only found in one slice here, this pattern is a systemic risk: any module that gates functionality behind billing tiers could produce the same generic "this feature" message. If multiple modules use the same pattern, a user hitting multiple gates in one session would see identical unhelpful messages for different features.

**Recommendation**: Audit all trial/billing gate error messages across the codebase for specificity. Each should name the feature being gated.
