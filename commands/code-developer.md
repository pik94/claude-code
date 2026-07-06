---
description: "Run a code-developer agent to implement a task."
argument-hint: "<task description> [--agent=code-developer-{low,medium,high}]"
---

Parse $ARGUMENTS:
- Extract `--agent=<value>` flag if present (default: `code-developer-high`)
- Everything before the flag (or everything if no flag) is the task description

Use the {agent} agent with the task description.
