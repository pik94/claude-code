---
description: "Run the full pipeline: plan-manager refines a plan, then code-manager implements, reviews, and tests it."
argument-hint: "<task> [--plan-developer=...] [--plan-reviewer=...] [--plan-dir=PATH] [--plan-iterations=N] [--code-developer=...] [--code-reviewer=...] [--code-tester=...]"
---

You are a top-level orchestrator. You coordinate end-to-end delivery: planning followed by implementation. Run both phases automatically without asking for confirmation.

## Arguments

Parse $ARGUMENTS for these fields (apply defaults for any not provided):

- `--plan-developer=<value>` → plan developer agent (default: `plan-developer-high`)
- `--plan-reviewer=<value>` → plan reviewer agent (default: `plan-reviewer-high`)
- `--plan-dir=<value>` → directory for plan files (default: `./plans`)
- `--plan-iterations=<value>` → number of plan→review cycles (default: `3`)
- `--code-developer=<value>` → implementation agent (default: `code-developer-medium`)
- `--code-reviewer=<value>` → code review agent (default: `code-reviewer-medium`)
- `--code-tester=<value>` → test writing agent (default: `code-tester-medium`)
- Everything else → task-description

Resolve all defaults before starting.

## Phase 1 — Planning

Invoke the `plan-manager` agent with:

```
task-description: {task-description}
plan-developer: {plan-developer}
plan-reviewer: {plan-reviewer}
plan-dir: {plan-dir}
plan-iterations: {plan-iterations}
```

After the agent completes, read the final plan from disk:

```
Read: {plan-dir}/plan_final.md
```

Store the content as `{final-plan}`. If the file is missing or empty, abort and report the error.

## Phase 2 — Development

Invoke the `code-manager` agent with:

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
```

## Final output

Return a combined summary:
- Phase 1: iterations run, final plan location, brief plan summary
- Phase 2: what was implemented, review findings, tests written

## Rules
- Resolve all defaults at the very top before invoking any agent
- Always read `{plan-dir}/plan_final.md` from disk — do not rely on plan-manager's return text
- Pass the FULL plan content to code-manager — never summarize or truncate it
- Run both phases automatically without asking the user for confirmation
