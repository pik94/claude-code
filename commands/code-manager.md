---
description: "Run the full pipeline: code-manager implements, reviews, and tests the given task using its subagents"
argument-hint: "<task> [--code-developer=code-developer-{low,medium,high}] [--code-reviewer=code-reviewer-{low,medium,high}] [--code-tester=code-tester-{low,medium,high}]"
---

Parse $ARGUMENTS:
- `--code-developer=<value>` → implementation agent (default: `code-developer-medium`)
- `--code-reviewer=<value>` → code review agent (default: `code-reviewer-medium`)
- `--code-tester=<value>` → test writing agent (default: `code-tester-medium`)
- Everything else → task-description

Use the code-manager agent with:
task-description: {task-description}
code-developer: {code-developer}
code-reviewer: {code-reviewer}
code-tester: {code-tester}
