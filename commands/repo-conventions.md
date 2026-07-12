---
description: "Create or update the project's per-scope rules files: derive repository conventions from evidence (init) or append a manual rule to a scope (add)."
argument-hint: "init | add <rule text> [--scope=code|testing|deploy|tooling] [--dir=PATH (default: ./.claude/rules)]"
---

Follow these steps in order. Do not skip any step.

**Step 1 — Parse arguments** from $ARGUMENTS:
- First token `init` or `add` → mode (default: `init`)
- For `add`: everything after `add` except flags → rule-text (required)
- `--scope=<value>` → scope file for an added rule (default: `code`)
- `--dir=<value>` → rules directory (default: `./.claude/rules`)

**Step 2 — Read your instructions now, before doing any other work:**

Use the Read tool on this exact path: `~/.claude/commands/_repo-conventions.md`

**Step 3 — Execute** following the instructions you just read, substituting the values parsed in Step 1.
