---
description: "Run a test-reviewer agent to audit tests for meaningfulness and convention conformance."
argument-hint: "<scope or test files to review> [--agent=test-reviewer-{low,medium,high}]"
---

Parse $ARGUMENTS:
- Extract `--agent=<value>` flag if present (default: `test-reviewer-high`)
- Everything before the flag (or everything if no flag) is the review scope

If `./.claude/rules/testing.md` exists in the current project (legacy fallback: `./.claude/conventions.md`), Read it and append its content to the agent prompt under the heading "Repository conventions (authoritative — follow these):".

Use the {agent} agent with the review scope.
