Fan-out audit: spawn one agent per small file batch, each writing structured findings to its own file. Three phases: pre-filter, individual deep-dives, cross-cutting pattern detection.

## YOUR ROLE AS ORCHESTRATOR

You are a mechanical dispatcher. Your job is:

1. Pre-filter files (one grep command, no iteration)
2. Group them into slices (simple directory grouping)
3. Show the plan to the user
4. Launch agents
5. Synthesize results

You are NOT an editor. Do NOT:

- Second-guess the slice count ("153 is too many" -- NO, that's fine)
- Re-filter after the first filter ("still too many" -- NO, show the plan)
- Merge slices to reduce count
- Exclude files because they "probably don't have relevant content"
- Make multiple filtering passes
- Skip Phase 2 for any reason
- Write synthesis "from memory" or "from agent return values in context"
- Proceed past a step if its output files don't exist

High slice counts (100-200) are EXPECTED and DESIRED. The whole point of fan-out is massive parallelism with small batches. A slice with 3 files is perfect. A slice with 1 file is fine. Show the plan, let the user decide.

**The value of this command is in the FILES it produces, not in what you summarize.** If agents complete but no files appear in `audits/{OUTPUT}/phase1/`, something is broken. STOP and debug.

## Usage

The user provides:

1. **TASK** - What to look for (the audit prompt each agent gets). If the task references an external doc (e.g., "check against style-guide.md"), read that doc and embed its full content in the agent prompt.
2. **OUTPUT** - A descriptive name for the output directory under `audits/`. e.g., `tropes-violations`, `refactoring-opportunities`, `error-handling-alignment`. Ask the user if not obvious from the task.
3. **TARGET** (optional) - Directory or list of directories to audit. Defaults to the current directory (`.`). Can be comma-separated: `TARGET=src/app,src/api,docs/`.

Example invocations:

- `/fan-out-audit find refactoring opportunities - duplicated logic, consolidation candidates, dead code, library alternatives`
- `/fan-out-audit check all user-facing text against ~/Downloads/tropes.md`
- `/fan-out-audit TARGET=src/app,src/api find components over 200 lines or with >3 props`

## Step 1: Parse input

Extract `TASK`, `OUTPUT`, and `TARGET` from the user's arguments: `$ARGUMENTS`

If `OUTPUT` is not specified or obvious from the task, ask the user what to name the output directory.

If `TARGET` is not specified, default to the current directory (`.`), excluding `node_modules`, `dist`, `.next`, `.turbo`, and other build artifacts.

If the TASK references an external document (file path or URL), read/fetch it now. Its content will be embedded directly into every agent's prompt so agents don't waste tokens re-reading it and all agents check against the exact same criteria.

## Step 2: Pre-filter to find relevant files

ONE pass. Run ONE grep/find command to get the file list. Do NOT iterate or re-filter.

The goal is to exclude files that obviously have zero relevance (e.g., binary files, type-only files, test fixtures). NOT to narrow down to a "manageable" number. If the filter returns 500 files, that's 500 files.

**How to pre-filter** (adapt the grep pattern to the TASK):

- For **text/copy audits** (tropes, positioning, tone): find files containing string literals, JSX text, or markdown prose. Exclude test files, `.d.ts`, and config.
  ```bash
  grep -rl --include="*.tsx" --include="*.ts" --include="*.md" --include="*.mdx" \
    -E '(["'"'"'`][\w\s]{15,}|<[a-z].*>[A-Z][\w\s]{10,})' {TARGET} \
    | grep -v '__tests__\|node_modules\|dist\|\.d\.ts' \
    | sort
  ```

- For **code pattern audits** (refactoring, duplication, error handling): find all code files. Skip pre-filtering if every code file is relevant.

- For **broad audits**: skip pre-filtering, use all source files.

**After the grep, MOVE ON.** Do not run a second filter. Do not count and say "too many." Whatever the grep returns is the file list.

## Step 3: Create slices

Group filtered files into slices. This is mechanical -- no judgment calls.

Rules:

1. **Keep files from the same directory together.**
2. **Max 8 files per slice.** If a directory has 30 files, split into 4 slices.
3. **Don't mix files from unrelated directories.**
4. **Name each slice** from the directory path: `components-billing`, `components-billing-2`, `docs-internal`.
5. **Single-file slices are fine.** Do not merge them with other directories.

Print the slice plan for the user:

```
Slice plan: 153 slices, 518 files total
Output: audits/tropes-violations/

  1. pages-ai (6 files): page.tsx, chat-panel.tsx, ...
  2. pages-analytics (2 files): page.tsx, chart.tsx
  ...
  153. docs-internal (8 files): architecture.md, setup.md, ...

Launch 153 Phase 1 agents? (y/n)
```

Wait for user confirmation. The user may say "go", "reduce to X", or "exclude Y". Follow their instruction.

## Step 4: Set up output directory

```bash
mkdir -p audits/{OUTPUT}/phase1 audits/{OUTPUT}/phase2
```

Save metadata:

- `audits/{OUTPUT}/TASK.md` - the full task description (including any embedded reference docs)
- `audits/{OUTPUT}/SLICE_PLAN.md` - the slice plan from Step 3

The output directory is inside the repo so the user can watch files appear in their editor as agents complete.

## Step 5: Build context payload (if needed)

Some tasks benefit from context embedded in every agent's prompt. Build this ONCE:

- **For text/copy audits**: the reference document (tropes doc, positioning doc, style guide). Embed verbatim.
- **For code audits**: a codebase map (directory tree + shared utility signatures). ~200 lines max.
- **For pattern audits**: the "gold standard" example to compare against.

This is pasted directly into the agent's prompt, NOT a separate file the agent reads.

## Step 6: Launch Phase 1 agents (individual slice deep-dives)

For each slice, spawn an agent with this prompt:

---

**Phase 1 agent prompt template** (fill in `{SLICE_FILES}`, `{TASK}`, `{CONTEXT_PAYLOAD}`, `{OUTPUT_FILE}`):

```
You are auditing a small batch of files. Read every file listed below and write detailed findings to an output file.

## Your files
{SLICE_FILES}
(listed as absolute paths, one per line)

## Your task
{TASK}

## Reference material
{CONTEXT_PAYLOAD}

## Instructions

1. Read EVERY file listed above. Do not skip any.
2. For each file, evaluate it against the task criteria.
3. Write your findings to: {OUTPUT_FILE}

## Output format

Write a markdown file with this EXACT structure. Use the EXACT category names and impact levels so findings can be programmatically grouped.

# Audit: {slice_name}
**Files inspected**: {count}
**Findings**: {count}

## Summary
1-3 sentences: what these files do and what you found.

## Findings

### Finding 1: {title}
- **File**: `path/to/file.ts:42`
- **Category**: {one of: duplication, consolidation, dead-code, library-alternative, pattern-violation, complexity, naming, missing-abstraction, inconsistency, tone, copy}
- **Impact**: {one of: high, medium, low}
- **Description**: What you found and why it matters. Be specific.
- **Suggestion**: Concrete fix. Name the specific change.
- **Evidence**:
  (the offending text or code, with enough context)

(repeat for each finding)

## No findings
If nothing noteworthy, say so with a 1-sentence explanation of what you checked. "No findings" is valid. Do NOT fabricate findings or pad with trivia.

## RULES
- Read EVERY file. Thoroughness is the entire point of this audit.
- Be specific: file paths with line numbers, exact quotes, concrete suggestions.
- Quality over quantity: 3 real findings beat 10 vague ones.
- Do NOT fabricate. Do NOT pad. "No findings" is respected.
- Use the EXACT category and impact names above. No synonyms.
```

---

**Execution**:

- Launch up to 10 agents per batch using `run_in_background: true`. Wait for each batch to complete before launching the next. (API rate limits are the practical bottleneck. 10 per batch is conservative and safe.)
- Use `subagent_type: "general-purpose"` and `model: "sonnet"`.
- Each agent writes to `audits/{OUTPUT}/phase1/{slice-name}.md`.

**CRITICAL: Agent type must be `general-purpose`, NOT `Explore`.** Explore agents do not have Write access and CANNOT create output files. This was tested and confirmed: Explore agents silently fail to write, producing zero output files. `general-purpose` agents have Write access and will create the output files as instructed.

**Safety note**: `general-purpose` agents also have Bash access. The agent prompt does NOT instruct them to run any commands, only to Read files and Write their output. But if safety is a concern, the user should run in default permission mode (not bypass) so Bash calls get prompted.

## Step 7: Verify Phase 1 output files exist

Before proceeding, count the files in `audits/{OUTPUT}/phase1/`. There should be one `.md` file per slice. If the count is zero or significantly less than the slice count, STOP and tell the user: "Phase 1 agents failed to write output files. Check agent type (must be general-purpose, not Explore)."

Do NOT skip this check. Do NOT proceed to Phase 2 or synthesis without Phase 1 files. Do NOT "synthesize from memory" or "from agent return values." The entire point is that findings are persisted as files.

## Step 8: Launch Phase 2 agents (cross-cutting pattern detection)

After ALL Phase 1 agents complete AND Phase 1 output files are verified, launch a second round.

Split Phase 1 output files into groups of ~12. For each group, spawn an agent:

---

**Phase 2 agent prompt template** (fill in `{FILE_LIST}`, `{TASK}`, `{OUTPUT_FILE}`):

```
You are a cross-cutting pattern detector. Read the Phase 1 audit findings below and identify patterns that span multiple file groups.

## Original task
{TASK}

## Your input files
Read ALL of these Phase 1 audit files:
{FILE_LIST}

## What to look for

1. **Same finding in multiple slices**: e.g., 5 slices all report "uses em dashes excessively" = systemic issue.
2. **Contradictory approaches**: Slice A does X, Slice B does Y for the same thing.
3. **Complementary findings**: Two findings that together suggest a bigger refactor.
4. **Systemic issues**: Patterns suggesting a codebase-wide problem.

## Output format

Write to: {OUTPUT_FILE}

# Cross-Cutting Patterns (Group {N})
**Slices analyzed**: {list}

## Pattern 1: {title}
- **Seen in**: {slices where this appears}
- **Category**: {same as Phase 1}
- **Combined impact**: {high|medium|low} with rationale
- **What's happening**: Description
- **Suggestion**: What a fix looks like
- **Estimated scope**: How many files/instances

(repeat for each pattern)

Only report patterns spanning 2+ slices. Do NOT repeat individual findings. If no cross-cutting patterns exist, say so.
```

---

**Execution**:

- Launch up to 10 agents per batch using `run_in_background: true`.
- Use `subagent_type: "general-purpose"`, `model: "opus"` (needs deeper reasoning).
- Write to `audits/{OUTPUT}/phase2/group-{N}.md`.

## Step 9: Synthesize

After ALL Phase 2 agents complete, read all Phase 2 files and a sampling of high-impact Phase 1 files.

**Do NOT skip Phase 2 and write synthesis from memory.** The synthesis MUST be based on the Phase 1 and Phase 2 FILES, not on agent return values you received in context. If Phase 2 files don't exist, go back to Step 8.

Write `audits/{OUTPUT}/SYNTHESIS.md`:

```markdown
# Fan-Out Audit Synthesis

**Task**: {TASK}
**Date**: {date}
**Slices audited**: {count}
**Total files inspected**: {sum}
**Total findings**: {sum}

## Top 10 Highest-Impact Opportunities

Ranked by impact. Cite Phase 1/Phase 2 files as evidence.

### 1. {title}

- **Impact**: ...
- **Evidence**: Found in {N} slices: {list with file references}
- **What to do**: Concrete action
- **Estimated scope**: {number of files/instances to fix}

## Cross-Cutting Patterns

Deduplicated and ranked from Phase 2.

## Slices with zero findings

{list with note: genuinely clean or did the agent phone it in?}

## Raw data

- Phase 1: `audits/{OUTPUT}/phase1/`
- Phase 2: `audits/{OUTPUT}/phase2/`
- Slice plan: `audits/{OUTPUT}/SLICE_PLAN.md`
- Task: `audits/{OUTPUT}/TASK.md`
```

Tell the user: "All findings in `audits/{OUTPUT}/`. Start with SYNTHESIS.md. Phase 1 has per-slice details, Phase 2 has cross-cutting patterns."

## Design principles

1. **The orchestrator is mechanical, not editorial.** Filter once, slice mechanically, show the plan, launch. No second-guessing.
2. **High slice counts are good.** 150 slices means 150 small, thorough inspections. That's the point.
3. **Small batches (5-8 files) so agents don't gloss over things.** AIs perform worse on large batches.
4. **Embed reference material in prompts, don't make agents fetch it.** Guarantees consistency.
5. **Show the slice plan before launching.** User can adjust. Orchestrator cannot.
6. **Fixed output format with exact category names.** Enables programmatic grouping in synthesis.
7. **Phase 2 uses Opus.** Cross-cutting pattern detection needs deeper reasoning than individual file inspection.
8. **Everything writes to files.** No information lost in agent-to-orchestrator summarization.
9. **Output in the repo, not /tmp.** User can watch files appear in real-time in their editor.
