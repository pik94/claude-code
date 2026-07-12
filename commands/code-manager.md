---
description: "Run the full dev pipeline: code-developer implements, parallel code reviews (auto-fix for critical issues), code-tester writes tests, test-reviewer audits them."
argument-hint: "<task> [--config=PATH] [--code-developer=...] [--code-reviewer=...] [--code-tester=...] [--test-reviewer=...] [--reviews=correctness,security,...] [--skip-review] [--skip-tests] [--skip-test-review]"
---

Follow these steps in order. Do not skip any step.

**Step 1 — Parse arguments** from $ARGUMENTS:
- `--config=<value>` → path to a JSON config file (optional)
- `--code-developer=<value>` → (default: `code-developer-high`)
- `--code-reviewer=<value>` → (default: `code-reviewer-high`)
- `--code-tester=<value>` → (default: `code-tester-high`)
- `--test-reviewer=<value>` → (default: `test-reviewer-high`)
- `--reviews=<value>` → comma-separated review types (default: `correctness,security,maintainability`; valid: `correctness`, `security`, `maintainability`, `performance`, `api-contract`, `docs`; `none` disables review)
- `--skip-review` → skip the review stage (default: false)
- `--skip-tests` → skip the testing and test-review stages (default: false)
- `--skip-test-review` → skip only the test-review stage (default: false)
- Everything else → task-description

**Resolution order for every field:** explicit flag > value from the `--config` file > value from `./.claude/pipeline.json` if it exists > default above. Read config files with the Read tool and parse as JSON; ignore keys you don't recognize; abort with an error if a config file passed via `--config` is missing or invalid JSON.

**Step 2 — Read your orchestration instructions now, before doing any other work:**

Use the Read tool on this exact path: `~/.claude/commands/_code-manager.md`

**Step 3 — Execute the pipeline** following the instructions you just read, substituting the resolved values from Step 1.
