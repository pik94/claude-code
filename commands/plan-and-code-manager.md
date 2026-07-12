---
description: "Run the full pipeline: plan-manager refines a plan, then code-manager implements, reviews, and tests it."
argument-hint: "<task> [--config=PATH] [--plan-developer=...] [--plan-reviewer=...] [--plan-dir=PATH] [--plan-iterations=N] [--code-developer=...] [--code-reviewer=...] [--code-tester=...] [--test-reviewer=...] [--reviews=...] [--skip-review] [--skip-tests] [--skip-test-review]"
---

You are a top-level orchestrator. You coordinate end-to-end delivery: planning followed by implementation. Both pipelines execute in THIS session — the plan-manager and code-manager instruction files are followed directly, never spawned as subagents (the Agent tool is unavailable to subagents). Run both phases automatically without asking for confirmation.

## Arguments

Parse $ARGUMENTS for these fields:

- `--config=<value>` → path to a JSON config file (optional)
- `--plan-developer=<value>` → plan developer agent (default: `plan-developer-high`)
- `--plan-reviewer=<value>` → plan reviewer agent (default: `plan-reviewer-high`)
- `--plan-dir=<value>` → directory for plan files (default: `./plans`)
- `--plan-iterations=<value>` → number of plan→review cycles (default: `3`)
- `--code-developer=<value>` → implementation agent (default: `code-developer-high`)
- `--code-reviewer=<value>` → code review agent (default: `code-reviewer-high`)
- `--code-tester=<value>` → test writing agent (default: `code-tester-high`)
- `--test-reviewer=<value>` → test review agent (default: `test-reviewer-high`)
- `--reviews=<value>` → comma-separated review types (default: `correctness,security,maintainability`; `none` disables review)
- `--skip-review` → skip the code review stage (default: false)
- `--skip-tests` → skip the testing and test-review stages (default: false)
- `--skip-test-review` → skip only the test-review stage (default: false)
- Everything else → task-description

**Resolution order for every field:** explicit flag > value from the `--config` file > value from `./.claude/pipeline.json` if it exists > default above. Read config files with the Read tool and parse as JSON; ignore keys you don't recognize; abort with an error if a config file passed via `--config` is missing or invalid JSON. Resolve all fields before starting.

## Phase 1 — Planning

Read `~/.claude/commands/_plan-manager.md` and execute that pipeline in this session with:

```
task-description: {task-description}
plan-developer: {plan-developer}
plan-reviewer: {plan-reviewer}
plan-dir: {plan-dir}
plan-iterations: {plan-iterations}
```

After the pipeline completes, read the final plan from disk:

```
Read: {plan-dir}/plan_final.md
```

Store the content as `{final-plan}`. If the file is missing or empty, abort and report the error.

## Phase 2 — Development

Read `~/.claude/commands/_code-manager.md` and execute that pipeline in this session with:

```
task-description:
You are implementing the following task based on a refined implementation plan.

## Original task
{task-description}

## Implementation plan
{final-plan}

Follow the plan as your primary implementation guide.

code-developer: {code-developer}
code-reviewer: {code-reviewer}
code-tester: {code-tester}
test-reviewer: {test-reviewer}
reviews: {reviews}
skip-review: {skip-review}
skip-tests: {skip-tests}
skip-test-review: {skip-test-review}
```

## Final output

Return a combined summary:
- Phase 1: iterations run, final plan location, brief plan summary
- Phase 2: what was implemented, review findings per review type, tests written, test review findings, stages skipped

## Rules
- Resolve all fields (explicit → config → `./.claude/pipeline.json` → defaults) at the very top before invoking any agent
- Never spawn plan-manager or code-manager as subagents — follow their `_*.md` instruction files in this session
- Always read `{plan-dir}/plan_final.md` from disk — do not rely on the planning phase's in-memory state
- Pass the FULL plan content to the development phase — never summarize or truncate it
- Run both phases automatically without asking the user for confirmation
- The final plan (`plan_final.md`) must be clean and self-contained — no references to iterations, reviews, reviewer feedback, or paragraph/section/point numbers. The planning pipeline is responsible for stripping this; verify it before passing the plan onward
- Code and test comments produced in Phase 2 must reference only the code and its (business) logic — never the task, the plan, plan step numbers, or reviews
