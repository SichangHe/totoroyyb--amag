# Plan Generation Checks

Run all three checks in order after writing `implementation_plan.md`.

---

## 1. Compliance Check

Verify all required sections exist:
```
grep_search("## TL;DR", implementation_plan.md)
grep_search("Must NOT Have", implementation_plan.md)
grep_search("## Test Strategy", implementation_plan.md)
grep_search("## Plan Consultant Summary", implementation_plan.md)
grep_search("## Implementation Steps", implementation_plan.md)
grep_search("QA Scenarios", implementation_plan.md)
grep_search("## Final Verification Wave", implementation_plan.md)
```
**If any search returns zero matches → fix the plan before proceeding.**

---

## 2. Post-Generation Self-Review

Re-read the plan and perform a gap-classification pass. Do this internally — do NOT present the raw checklist to the user:

```
□ Every task has concrete, agent-executable acceptance criteria (commands, not human checks)?
□ All file references exist in the codebase (grep/find evidence)?
□ No assumptions about business logic are baked in without evidence?
□ Guardrails from the consultant review are incorporated?
□ Scope boundaries clearly defined — at least 2-3 "Must NOT Have" items?
□ Every task has QA Scenarios: 1 happy path + 1 failure case minimum?
□ No vague terms ("fast", "clean", "robust") without concrete definitions?
```

For each gap found, classify and act:

| Class | Definition | Action |
|---|---|---|
| **CRITICAL** | Requires user decision — business logic, unclear requirement | Insert `[DECISION NEEDED: {description}]` placeholder in plan → surface in Step 8 summary |
| **MINOR** | Self-resolvable — missing file ref findable via search | Fix silently in plan. Note in `## Plan Consultant Summary` |
| **AMBIGUOUS** | Has a reasonable default — naming style, error handling | Apply default. Disclose in plan under "Defaults Applied" |

**CRITICAL gaps**: Do NOT proceed to Step 8 without surfacing them. Present via `notify_user` alongside the High-Accuracy Gate options, so the user can answer decisions and choose critic review in one round-trip.

**MINOR / AMBIGUOUS only**: Fix silently, then proceed to Step 8 immediately.

---

## 3. Quality Standards

| Standard | Requirement |
|---|---|
| Task count | 3-8 per wave; final wave always present |
| QA scenarios | 1 happy path + 1 failure case per task minimum |
| Acceptance criteria | Agent-executable commands only (no human checks) |
| File references | Specific `file:line` citations discovered in exploration |
| No vague terms | "fast" → "p99 < 200ms"; "clean" → "follows patterns in `src/utils/`" |
| Scope guardrails | At least 2-3 explicit "Must NOT Have" items |
