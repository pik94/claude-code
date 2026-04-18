---
description: "Run a code-reviewer agent to audit code for bugs and issues."
argument-hint: "<scope or files to review> [--agent=code-reviewer-{low,medium,high}]"
---

Parse $ARGUMENTS:
- Extract `--agent=<value>` flag if present (default: `code-reviewer-medium`)
- Everything before the flag (or everything if no flag) is the review scope

Use the {agent} agent with the review scope.
