---
description: "Run a code-tester agent to write unit tests."
argument-hint: "<target files or feature> [--agent=code-tester-{low,medium,high}]"
---

Parse $ARGUMENTS:
- Extract `--agent=<value>` flag if present (default: `code-tester-high`)
- Everything before the flag (or everything if no flag) is the target description

Use the {agent} agent with the target description.
