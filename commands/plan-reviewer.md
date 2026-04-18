---
description: "Run a plan-reviewer agent to critique an implementation plan."
argument-hint: "<plan or task to review> [--agent=plan-reviewer-{low,medium,high}]"
---

Parse $ARGUMENTS:
- Extract `--agent=<value>` flag if present (default: `plan-reviewer-high`)
- Everything before the flag (or everything if no flag) is the content to review

Use the {agent} agent with the content.
