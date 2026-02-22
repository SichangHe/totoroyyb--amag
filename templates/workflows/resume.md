---
description: Resume work from a previous plan — self-validates progress and continues
---

# /resume — Cross-Session Resume

Pick up where you left off. Reads `.amag/active-plan.md`, validates actual progress via checkboxes, and resumes from the first uncompleted task.

## Progress Tracking

Call `task_boundary` at **every step transition** with:
- **TaskName**: `"Resuming: {plan name}"`
- **Mode**: `EXECUTION`
- **TaskStatus**: Use the directive shown at each step (e.g. `"Step 2/4: Validating progress"`)
- **TaskSummary**: Cumulative — plan state, progress found, context rebuilt

## Steps

### 1. Read Active Plan
<!-- task_boundary: TaskStatus="Step 1/4: Reading active plan" -->

Read `.amag/active-plan.md` in the project root.

- **Not found** → "No active plan. Use `/plan` to create one."
- **Found** → proceed to self-validation

### 2. Self-Validate (Ignore Stale Metadata)
<!-- task_boundary: TaskStatus="Step 2/4: Validating progress checkboxes" -->

Parse the file content. Count checkboxes:

| Checkbox | Meaning |
|---|---|
| `- [ ]` | Uncompleted task |
| `- [x]` or `- [X]` | Completed task |

Calculate: `completed / total`

**Fix inconsistencies automatically:**
- All checked but `status` ≠ `completed` → update status to `completed`, inform user
- Some unchecked but `status` = `completed` → warn user: "Status says complete but X tasks remain unchecked"

**If all tasks are complete:**
- "Previous plan [name] is complete (X/X tasks). Use `/plan` to start new work."
- Stop here

### 3. Rebuild Context
<!-- task_boundary: TaskStatus="Step 3/4: Rebuilding context from artifacts" -->

1. Read `.amag/active-plan.md` for task list and progress
2. Search for an `implementation_plan.md` artifact (if in the same conversation that created it)
3. Read `.amag/notepads/{plan-name}.md` if it exists — it contains learnings accumulated during previous task execution
4. If detailed plan not available (new conversation): the checklist in `active-plan.md` is sufficient to continue
5. Create `task.md` artifact in this conversation, seeded from `active-plan.md` checkboxes

### 4. Begin Execution
<!-- task_boundary: TaskStatus="Step 4/4: Beginning execution" -->

1. Set `status: in-progress` and `last_updated` in `.amag/active-plan.md`
2. Set `task_boundary` with first uncompleted task
3. Execute following `/start-work` protocol from Step 3 onwards

All progress updates follow the **dual-write protocol**: update `.amag/active-plan.md`, `task.md` artifact, AND `task_boundary` simultaneously.

## When to Use

- User says "resume", "continue", or uses `/resume`
- Starting a new conversation and wanting to continue previous work
- After a session interruption or crash
