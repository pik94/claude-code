---
description: "Create a JSON pipeline config with default values, usable by /plan-manager, /code-manager, and /plan-and-code-manager."
argument-hint: "[path] (default: ./.claude/pipeline.json)"
---

Create a pipeline config file pre-filled with default values. Follow these steps in order.

**Step 1 — Resolve the path** from $ARGUMENTS: the first non-flag token is the target path (default: `./.claude/pipeline.json`).

**Step 2 — Guard against overwrite.** If the file already exists: Read it, show its current content, and stop — do NOT overwrite. Tell the user to edit it directly, or delete it and re-run this command.

**Step 3 — Write the config.** Create the parent directory if needed, then write exactly this content:

```json
{
  "plan-developer": "plan-developer-medium",
  "plan-reviewer": "plan-reviewer-medium",
  "plan-dir": "./plans",
  "plan-iterations": 3,
  "code-developer": "code-developer-medium",
  "code-reviewer": "code-reviewer-medium",
  "code-tester": "code-tester-medium",
  "test-reviewer": "test-reviewer-medium",
  "reviews": ["correctness", "security", "maintainability"],
  "skip-review": false,
  "skip-tests": false,
  "skip-test-review": false
}
```

**Step 4 — Explain.** Report the created file path and briefly explain:
- One config serves all three orchestrators — each command reads only the keys it knows and ignores the rest.
- Resolution order for every parameter: explicit `--flag` > file passed via `--config=PATH` > auto-discovered `./.claude/pipeline.json` > built-in default.
- Valid `"reviews"` values: `correctness`, `security`, `maintainability`, `performance`, `api-contract`, `docs`. To skip stages: `"skip-review": true` (or `"reviews": []`), `"skip-tests": true`, `"skip-test-review": true`.
