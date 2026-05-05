---
description: "Run the full dev pipeline: code-developer implements, code-reviewer audits (auto-fix for critical issues), code-tester writes tests."
argument-hint: "<task> [--code-developer=code-developer-{low,medium,high}] [--code-reviewer=code-reviewer-{low,medium,high}] [--code-tester=code-tester-{low,medium,high}]"
---

Follow these steps in order. Do not skip any step.

**Step 1 — Parse arguments** from $ARGUMENTS:
- `--code-developer=<value>` → (default: `code-developer-medium`)
- `--code-reviewer=<value>` → (default: `code-reviewer-medium`)
- `--code-tester=<value>` → (default: `code-tester-medium`)
- Everything else → task-description

**Step 2 — Read your orchestration instructions now, before doing any other work:**

Use the Read tool on this exact path: `~/.claude/commands/_code-manager.md`

**Step 3 — Execute the pipeline** following the instructions you just read, substituting the values parsed in Step 1.
