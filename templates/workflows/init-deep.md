---
description: Generate hierarchical GEMINI.md context files across the codebase
---

# /init-deep — Deep Codebase Initialization

Generate hierarchical GEMINI.md documentation files across the entire codebase. These files serve as AI-readable context that helps agents understand each directory's purpose, key files, and working instructions.

## Progress Tracking

Call `task_boundary` at **every step transition** with:
- **TaskName**: `"Init Deep: {project}"`
- **Mode**: `EXECUTION`
- **TaskStatus**: Use the directive shown at each step (e.g. `"Step 3/5: Generating GEMINI.md — Level [N]"`)
- **TaskSummary**: Cumulative — directories mapped, files generated

## Steps

### 1. Map Directory Structure
<!-- task_boundary: TaskStatus="Step 1/5: Mapping directory structure" -->

```
find_by_name with Pattern="*" and Type="directory"
```

Exclude: `node_modules`, `.git`, `dist`, `build`, `__pycache__`, `.venv`, `coverage`, `.next`

### 2. Plan Generation Order
<!-- task_boundary: TaskStatus="Step 2/5: Planning generation order" -->

Generate **parent levels before child levels** to ensure references are valid:

```
Level 0: / (root)
Level 1: /src, /docs, /tests
Level 2: /src/components, /src/utils
```

### 3. Generate GEMINI.md Per Directory
<!-- task_boundary: TaskStatus="Step 3/5: Generating GEMINI.md files" -->

For each directory:
1. Read key files using `view_file_outline` (don't read entire large files)
2. Analyze purpose and relationships
3. Write GEMINI.md with this template:

```markdown
# {Directory Name}

## Purpose
{One paragraph: what this directory contains and its role}

## Key Files
| File | Description |
|------|-------------|
| `file.ts` | Brief description |

## Subdirectories
| Directory | Purpose |
|-----------|---------|
| `subdir/` | What it contains |

## For AI Agents

### Working Here
{Special instructions for modifying files in this directory}

### Testing
{How to verify changes in this directory}

### Patterns
{Code conventions used here}
```

### 4. Skip Rules
<!-- task_boundary: TaskStatus="Step 4/5: Applying skip rules" -->

| Condition | Action |
|-----------|--------|
| Empty directory (no files, no subdirs) | Skip |
| Only generated/build files (.min.js, .map) | Skip |
| Only config files | Create minimal GEMINI.md |
| Has substantive code | Full GEMINI.md |

### 5. Validate
<!-- task_boundary: TaskStatus="Step 5/5: Validating generated files" -->

After generation:
- Verify all GEMINI.md files reference real files
- Ensure no orphaned GEMINI.md for deleted directories
- Check that descriptions are specific, not generic boilerplate

## When to Use

- User says "init-deep", "generate context files", or uses `/init-deep`
- First encounter with a large codebase — generating persistent AI-readable context
- After significant structural changes to the codebase (new modules, directory restructuring)

## When NOT to Use

- Codebase already has GEMINI.md files and hasn't changed structurally
- Quick exploration of a codebase (use `/explore` instead — read-only, no file generation)
- Understanding a single module (use Exploratory inline search)
