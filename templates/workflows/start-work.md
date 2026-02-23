---
description: Execute an implementation plan task-by-task with category-aware delegation
---

# /start-work — Plan Execution

Read an existing plan and execute it task-by-task with verification. This is the execution counterpart to `/plan`.

**You are a conductor, not a player.** For each task, adopt the right persona (category), load relevant skills, verify completion, then move to the next task.

## Progress Tracking

Call `task_boundary` at **every step transition** with:
- **TaskName**: `"Executing: {plan name}"`
- **Mode**: `EXECUTION` (switch to `VERIFICATION` at Step 5)
- **TaskStatus**: Use the directive shown at each step. During Step 3, update per-task: `"Step 3/5: Task [N/total] — {task title}"`
- **TaskSummary**: Cumulative — tasks completed, issues found, learnings

## Steps

### 1. Find and Read the Plan
<!-- task_boundary: TaskStatus="Step 1/6: Reading plan" -->

**Check both sources and combine:**

1. Read `.amag/active-plan.md` in the project root (cross-session truth for checkbox state)
2. Check for an `implementation_plan.md` artifact in this conversation (detailed plan with file references, acceptance criteria)

**If `.amag/active-plan.md` exists:**
- **Self-validate** — parse all `- [ ]` and `- [x]` checkboxes
- Calculate actual progress: `completed / total`
- If all tasks checked → inform user: "Plan is complete. Use `/plan` for new work."
- If some unchecked → proceed with resume
- If `implementation_plan.md` artifact also exists → use it for detailed context per task

**If only `implementation_plan.md` artifact exists (no active-plan.md):**
- This means the plan was approved but Step 10 of `/plan` wasn't executed, or this is the first `/start-work` in the same conversation
- Create `.amag/active-plan.md` from the artifact's task list before proceeding

**If neither found:**
- Tell the user: "No active plan found. Use `/plan` to create one."

### 2. Create Task Breakdown
<!-- task_boundary: TaskStatus="Step 2/6: Creating task breakdown" -->

Create a `task.md` artifact with all tasks from the plan. Copy checkbox state from `.amag/active-plan.md` if resuming.

Update `task_boundary` to reflect current state.

Update `.amag/active-plan.md` YAML header:
- Set `status: in-progress`
- Set `last_updated` to current timestamp

### 3. Execute Task-by-Task
<!-- task_boundary: TaskStatus="Step 3/6: Task [N/total] — {task title}" -->

For each uncompleted task:

#### a) Classify the Category

Before executing, classify what kind of work this task requires:

| Category | When to Activate | Persona Shift |
|---|---|---|
| **visual** | UI, CSS, frontend, design, animation | Design-first: bold aesthetics, distinctive typography, cohesive palettes |
| **deep** | Complex logic, architecture, multi-file refactor | Autonomous: explore extensively (5-10 files minimum), build full mental model, act decisively |
| **quick** | Single file, typo fix, small change | Efficient: fast, no over-engineering, minimal overhead |
| **writing** | Documentation, README, comments, tech writing | Anti-slop: no "delve"/"leverage", plain words, human tone, varied sentences |
| **general** | Everything else | Standard execution with full verification |

#### b) Load Relevant Skills

Check if any installed skills apply (workflow-defined loading — see GEMINI.md Skill Loading Protocol):
- Frontend/UI work → activate `frontend-ui-ux` skill
- Git operations → activate `git-master` skill
- Browser testing needed → activate `browser-testing` skill
- Deep exploration → activate `deep-work` skill
- Writing/docs → activate `writing` skill

#### c) Execute with Category Persona

Adopt the category's mindset for the duration of this task. Match existing codebase patterns. Implement exactly what the plan specifies.

#### d) Static Analysis — Per Task (MANDATORY, before marking done)

After editing any files for this task, run targeted static analysis on those specific files — do not wait for the final verification wave:

1. **Detect toolchain** from config files: `tsconfig.json` → `tsc --noEmit`, `mypy.ini`/`pyproject.toml` → `mypy <file>`, `Cargo.toml` → `cargo check`, `.eslintrc`/`biome.json` → `eslint <file>`.
2. **Run targeted check** on only the files you changed in this task. This is fast — not a full project build.
3. **Zero new errors required** — if errors appear, fix them before proceeding to `e)`.

This is the AMAG equivalent of OMO's `lsp_diagnostics` — run it once per task, not once per plan.

**Anti-pattern**: Do NOT defer this to the final verification wave. By then, errors from earlier tasks compound and are harder to trace.

#### e) Verify Against Acceptance Criteria

Check the plan's acceptance criteria for this task:
- Run build/test commands if the plan specifies them for this task
- Verify with `grep_search` for expected patterns
- Use `browser_subagent` for visual verification if needed

Mark task as completed only AFTER both `d)` (static analysis clean) and `e)` (acceptance criteria) pass.

#### f) Dual-Write Progress (MANDATORY after each task)

All three updates must happen together:

1. **`.amag/active-plan.md`** — mark `[x]` on the completed task, update `last_updated` in YAML header
2. **`task.md` artifact** — mark `[x]` on the completed task
3. **`task_boundary`** — update TaskStatus and TaskSummary

**Never update one without the others.**

#### g) Accumulate Learnings

After each task, write key findings to `.amag/notepads/{plan-name}.md`. Append — never overwrite.

```markdown
## [Task N: title]
- Convention: [naming pattern, style convention discovered]
- Gotcha: [edge case, unexpected behavior encountered]
- Decision: [trade-off made, alternative considered and rejected]
```

**Before starting each subsequent task**: re-read this file and apply accumulated knowledge. This prevents repeating mistakes and ensures consistency across tasks.

### 4. Final Verification (NON-NEGOTIABLE — after ALL tasks)
<!-- task_boundary: Mode=VERIFICATION, TaskStatus="Step 4/5: Running final verification" -->

Mandatory catch-all — runs GEMINI.md Verification Protocol (Steps 1-6) across ALL files modified during the entire plan, PLUS these plan-specific checks:

#### a) Scope Fidelity — Plan vs Reality

Self-verify each plan task against actual changes (you already know the plan):

- For each task in the plan: verify it was implemented completely
- Check: nothing was built that the plan didn't specify (scope creep)
- Check: nothing was skipped or simplified ("basic version")
- Check: "Must NOT Have" guardrails from the plan were respected
- Output: `Tasks [N/N compliant] | VERDICT: APPROVE/REVISE/REJECT`

If REVISE or REJECT: fix all flagged blocking issues before proceeding to step `b)`. If REVISE, re-run scope fidelity check after fixes. If REJECT, surface to user via `notify_user`.

#### b) Acceptance Criteria

- Verify EVERY acceptance criterion from the plan with tool-produced evidence
- Run QA scenarios from the plan: follow exact steps, capture evidence
- No criterion passes on assertion alone — run the command, show the output
- Output: `Criteria [N/N pass] | QA Scenarios [N/N pass] | Evidence [CAPTURED]`

#### c) Code Quality

Activate `architecture-advisor` skill for review. Check all changed files for: `as any`/`@ts-ignore`, empty catches, `console.log` in production code, commented-out code, unused imports. Check AI slop per `code-quality.md` Section 6.

#### d) Report
- Summarize what was implemented, verified, and any issues encountered
- Re-read the original plan one final time — did you miss anything?

#### e) Stuck During Verification

If a build or test command hangs, follow `error-recovery.md` Long-Running Command Protocol:

1. **Poll at 30s intervals**. Zero output growth for 2 polls = hung.
2. **Kill it** — `send_command_input(CommandId, Terminate=true)`.
3. **Mark the task blocked** — annotate as `[blocked]` in `task.md` and `.amag/active-plan.md`.
4. **Notify the user** via `notify_user` with hung command details and partial output.
5. **Do NOT mark the plan complete** while any task remains unverified.

### 5. Mark Plan Complete
<!-- task_boundary: TaskStatus="Step 5/5: Marking plan complete" -->

1. Update `.amag/active-plan.md` YAML header: `status: completed`, `last_updated`
2. Create `walkthrough.md` artifact summarizing what was implemented and verified
3. Do NOT delete `active-plan.md` — leave it as a completed record

## Resume Support

If the session is interrupted mid-work:

- `.amag/active-plan.md` checkboxes reflect exactly which tasks are done
- A new session can read this file, self-validate, and resume at the first unchecked task
- No external state files or session IDs needed — the checkboxes are the truth

## When to Use

- After `/plan` generates and user approves an implementation plan
- User says "start work", "execute the plan", or "start-work"
- When there's an existing plan to execute (in conversation or `.amag/active-plan.md`)
