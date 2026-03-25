# Phase 2: Cross-Cutting Patterns (Group 4)

**Slices analyzed**: 12 (`apps/api/src/application/` triage, voice, work-items, workflows; `apps/api/src/domain/` attachments, entities, knowledge, user; `apps/api/src/infrastructure/` queues; `apps/api/src/integrations/` core, errors, platforms-1)

---

## Pattern 1: "Please reconnect your account" as formulaic filler across all platform error files

**Slices affected**: integrations-platforms-1 (4 of 4 platforms in batch)

Every platform error file in this batch (Asana, Facebook Messenger, GitHub, Google Calendar) repeats the pattern `"[Problem]. Please reconnect your account."` across multiple error factories (credentials invalid, integration not found, token expired, unauthorized). The word "Please" is deferential filler that adds no information, and the instruction is mechanically identical regardless of the actual problem.

Total instances in this batch alone: 10+ across 4 platforms. This is the dominant trope pattern in the integration layer and will recur in Groups 5's platform batches.

**Suggested fix**: Drop "Please" from all reconnect messages. Use direct imperatives: `"Reconnect your account."` or `"Reconnect your account to continue."` Apply this as a single search-and-replace across all platform error files.

---

## Pattern 2: "Please try again later" as vague rate-limit filler

**Slices affected**: integrations-platforms-1 (3 of 4 platforms: Facebook Messenger, GitHub, Google Calendar)

Three of four platforms use `"Please try again later."` as the fallback when `retryAfter` information is unavailable. This phrase provides zero actionable information and is one of the most overused stock phrases in AI-generated error copy.

**Suggested fix**: Either state the fact without instruction (`"[Platform] API rate limited."`) or surface retry timing when available (`"Rate limited. Retry after ${retryAfterSeconds}s."`). Remove "Please try again later" entirely.

---

## Pattern 3: Sycophantic tone in triage error messages

**Slices affected**: triage (1 of 12)

The triage module contains `"Congratulations! You're already at triage - no items to process."` -- an unprompted celebratory exclamation in an error message. This is the only instance of sycophantic language found across all 12 slices in this group, making it an outlier rather than a systemic pattern. However, it is the most egregious single violation in the group.

**Suggested fix**: `"No items to process."`

---

## Pattern 4: Inconsistent trailing period usage across platform error messages

**Slices affected**: integrations-platforms-1 (2 of 4 platforms: Asana, Facebook Messenger)

Some messages within the same file end with a period, others do not, with no discernible rule. For example, `"Asana integration is not active."` (period) vs `"Asana API returned an error"` (no period). This inconsistency signals mechanically generated text rather than deliberate editing.

**Suggested fix**: Pick one convention (no trailing periods on error messages is cleaner) and apply uniformly across all platform error files.

---

## Pattern 5: Clean backend infrastructure across domain and application layers

**Slices affected**: voice, work-items, workflows, attachments, entities, knowledge, user, queues, integrations-core, integrations-errors (10 of 12)

Ten of twelve slices had zero findings. The domain layer, infrastructure layer, and non-platform integration files are consistently clean: terse error codes, factual messages, no AI writing tropes. The trope problems are concentrated entirely in the platform-specific integration error files and one application module (triage).
