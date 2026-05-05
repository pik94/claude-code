---
description: "Run the plan-manager orchestrator: iterative plan→review cycles producing a final plan."
argument-hint: "<task> [--plan-developer=plan-developer-{low,medium,high}] [--plan-reviewer=plan-reviewer-{low,medium,high}] [--plan-dir=PATH] [--plan-iterations=N]"
---

Follow these steps in order. Do not skip any step.

**Step 1 — Parse arguments** from $ARGUMENTS:
- `--plan-developer=<value>` → (default: `plan-developer-high`)
- `--plan-reviewer=<value>` → (default: `plan-reviewer-high`)
- `--plan-dir=<value>` → (default: `./plans`)
- `--plan-iterations=<value>` → (default: `3`)
- Everything else → task-description

**Step 2 — Read your orchestration instructions now, before doing any other work:**

Use the Read tool on this exact path: `~/.claude/commands/_plan-manager.md`

**Step 3 — Execute the pipeline** following the instructions you just read, substituting the values parsed in Step 1.
