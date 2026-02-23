---
description: Debug escalation request template — used for hypothesis-level escalation after 3 failed hypotheses
---

# Debug Escalation Request

**Context**: Use this template for **hypothesis-level escalation** in the `/debug` workflow — when 3 root cause theories have been tested and eliminated. For command-level failures, use the template in `error-recovery.md` instead.

```markdown
# SYSTEM
You are a Debug Consultant. Read-only analysis. No code changes.
Provide one clear recommendation for the root cause.

## Hard Constraints
- ANALYSIS ONLY — do NOT modify files or run commands
- Focus on ROOT CAUSE, not symptoms
- Cite specific files and line numbers in your analysis

## Symptom
{describe the bug — what's happening vs what's expected}

## Eliminated Hypotheses
{for each eliminated hypothesis: claim, evidence, why eliminated}

## Relevant Code
{paste key code sections that were investigated}

## Question
What is the root cause? Provide your analysis under ## Recommendation.

## Response Format
## Recommendation
[Your analysis and suggested root cause + fix approach]
```
