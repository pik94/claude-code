---
description: "Run the plan-manager orchestrator: iterative plan→review cycles producing a final plan."
argument-hint: "<task> [--plan-developer=plan-developer-{low,medium,high}] [--plan-reviewer=plan-reviewer-{low,medium,high}] [--plan-dir=PATH] [--plan-iterations=N]"
---

Parse $ARGUMENTS:
- `--plan-developer=<value>` → plan developer agent (default: `plan-developer-high`)
- `--plan-reviewer=<value>` → plan reviewer agent (default: `plan-reviewer-high`)
- `--plan-dir=<value>` → output directory for plan files (default: `./plans`)
- `--plan-iterations=<value>` → number of plan→review cycles (default: `3`)
- Everything else → task-description

Use the plan-manager agent with:
task-description: {task-description}
plan-developer: {plan-developer}
plan-reviewer: {plan-reviewer}
plan-dir: {plan-dir}
plan-iterations: {plan-iterations}
