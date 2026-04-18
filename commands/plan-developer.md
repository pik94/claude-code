---
description: "Run a plan-developer agent to create an implementation plan."
argument-hint: "<task description> [--agent=plan-developer-{low,medium,high}]"
---

Parse $ARGUMENTS:
- Extract `--agent=<value>` flag if present (default: `plan-developer-high`)
- Everything before the flag (or everything if no flag) is the task description

Use the {agent} agent with the task description.
