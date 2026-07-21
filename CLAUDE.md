# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is the global Claude Code configuration directory (`~/.claude`), version-controlled to track reusable agents, slash commands, skills, and settings across all projects.

The `.gitignore` excludes secrets (`.credentials.json`, `settings.local.json`) and runtime data (`sessions/`, `history.jsonl`, `projects/`, `file-history/`, `session-env/`).

## Tracked contents

- `agents/` — subagent definitions (`.md` files with YAML frontmatter)
- `commands/` — slash command definitions and agent instruction files (`.md` files)
- `skills/` — installed skill packages (each in its own subdirectory with a `SKILL.md`)
- `settings.json` — global permissions, model, terminal, and safety config

## Agent architecture

Agents come in three tiers (`-low` = haiku, `-medium` = sonnet, `-high` = opus) plus orchestrators:

```
Explore                  — fast codebase search (Haiku); finds files, searches code
code-developer-{tier}    — implements tasks, runs linters/tests, commits
code-reviewer-{tier}     — audits code for bugs/crashes, read-only
code-tester-{tier}       — writes unit tests following existing project patterns
test-reviewer-{tier}     — audits tests for tautology/triviality and convention conformance, read-only
plan-developer-{tier}    — writes structured implementation plans (no code changes)
plan-reviewer-{tier}     — critiques plans, returns structured feedback (no code changes)

plan-manager             — orchestrates: plan-developer ⇄ plan-reviewer × N → plan_final.md
code-manager             — orchestrates: code-developer → parallel reviews (merged fix if critical) → code-tester → test-reviewer (fix if critical)
plan-and-code-manager    — orchestrates: plan-manager → code-manager end-to-end
```

**Explore agent:** `code-developer`, `code-reviewer`, and `plan-developer` agents delegate broad codebase searches to `Explore` (Haiku) instead of running Glob/Grep themselves. Pass a description plus thoroughness level (`quick` / `medium` / `very thorough`). Use Read directly only for known paths.

**Key invariant:** agents share no memory. Orchestrators must pass the full text of prior outputs when invoking the next agent — never rely on return text alone; read saved files from disk.

**Handoff convention:** `plan-manager` writes its canonical output to `{plan-dir}/plan_final.md`. `plan-and-code-manager` reads this file from disk before invoking `code-manager`.

**Fix loop:** `code-manager` triggers at most one `code-developer` fix pass (critical issues merged from ALL parallel reviews into a single list — never one pass per review) and at most one `code-tester` fix pass (critical test-review issues). Only **critical** issues trigger a pass (crashes, data loss, security, tautological/failing tests); warnings and style issues do not.

**Parallel reviews:** `code-manager`'s review stage runs a configurable set of review types concurrently (one `code-reviewer` agent per type, launched in a single message): `correctness`, `security`, `maintainability` (defaults), plus opt-in `performance`, `api-contract`, `docs`. Focus checklists live in `_code-manager.md`; the reviewer honors a "Review focus" block in its arguments.

## Pipeline configuration

Orchestrator parameters can come from a JSON config instead of flags. Resolution order for every field: explicit `--flag` > file passed via `--config=PATH` > auto-discovered `./.claude/pipeline.json` in the project > built-in default. One config serves all three orchestrators; each reads only the keys it knows and ignores the rest. `/pipeline-config-init [path]` creates the config with default values.

Keys: `plan-developer`, `plan-reviewer`, `plan-dir`, `plan-iterations`, `code-developer`, `code-reviewer`, `code-tester`, `test-reviewer`, `reviews` (array), `skip-review`, `skip-tests`, `skip-test-review`.

**Repository rules:** `/repo-conventions init` derives project-specific rules from repo evidence into per-scope files in the project's `./.claude/rules/` — `code.md`, `testing.md`, `deploy.md`, `tooling.md` (only scopes with evidence are created); `/repo-conventions add <rule> --scope=<scope>` appends a manual rule to one scope file (`## Manual rules` sections are never auto-edited). `code-manager` routes each rules file only to the agents it applies to: `code.md` → code-developer + maintainability review, `testing.md` → code-tester + test-reviewer, `deploy.md` → code-developer, `tooling.md` → code-developer + code-tester, unrecognized scopes → code-developer. Legacy fallback: a single `./.claude/conventions.md` is injected whole into developer/tester/test-reviewer prompts.

## Slash commands

Commands without a `_` prefix are user-facing (become `/command-name`). Files with a `_` prefix are instruction files loaded by agents — not directly invokable.

| Command | Delegates to | Key flags |
|---|---|---|
| `/code-developer` | `code-developer-high` | `--agent=code-developer-{low,medium,high}` |
| `/code-reviewer` | `code-reviewer-high` | `--agent=code-reviewer-{low,medium,high}` |
| `/code-tester` | `code-tester-high` | `--agent=code-tester-{low,medium,high}` |
| `/test-reviewer` | `test-reviewer-high` | `--agent=test-reviewer-{low,medium,high}` |
| `/plan-developer` | `plan-developer-medium` | `--agent=plan-developer-{low,medium,high}` |
| `/plan-reviewer` | `plan-reviewer-medium` | `--agent=plan-reviewer-{low,medium,high}` |
| `/code-manager` | `code-manager` | `--config=`, `--code-developer=`, `--code-reviewer=`, `--code-tester=`, `--test-reviewer=`, `--reviews=`, `--skip-review`, `--skip-tests`, `--skip-test-review` |
| `/plan-manager` | `plan-manager` | `--config=`, `--plan-developer=`, `--plan-reviewer=`, `--plan-dir=`, `--plan-iterations=` |
| `/plan-and-code-manager` | `plan-and-code-manager` | all of the above; `--plan-dir` defaults to `./plans` |
| `/pipeline-config-init` | (runs inline) | `[path]` — creates a default JSON pipeline config (default: `./.claude/pipeline.json`) |
| `/repo-conventions` | (runs inline; `init` uses `Explore`) | `init` \| `add <rule>`, `--scope=code\|testing\|deploy\|tooling`, `--dir=` (default: `./.claude/rules`) |

## Settings

`settings.json` sets `defaultMode: auto` and pre-allows common tools (git, uv, make, docker, python3, pip, npm, node, ls, cat, find, grep, Agent) so they never prompt. Destructive commands (`rm`, `docker rm`, `git reset`) require confirmation.

## Conventions when adding agents or commands

- Agent frontmatter must include `name`, `description`, and `tools` (restrict to minimum needed). Add `model` only when the tier-based naming doesn't apply.
- Planner-type agents must state in their rules: they may not modify source code, only write markdown files.
- User-facing command files have a YAML frontmatter block with `description` and `argument-hint`. Instruction files (prefixed `_`) have no frontmatter — they are raw prompt text.
- Orchestrators resolve all optional args to their defaults at the very top before invoking any subagent.
- Output directories for each pipeline stage must be distinct (never share `plan-dir` and dev output dir).
- Pipeline output files follow the pattern: `{dir}/plan_v{n}.md`, `{dir}/review_v{n}.md`, `{dir}/plan_final.md`.

## Agent instruction authoring

Agent body (after frontmatter) must begin with an explicit mandatory Read directive, then `$ARGUMENTS`:

```
**Read your instructions before doing anything else.**

Use the Read tool now:
`~/.claude/commands/_<name>.md`

Do not start any other work until you have read that file.

$ARGUMENTS
```

The `commands/_*.md` files are the single source of truth for instruction prose and are shared across agent tiers. To update instructions for a group of agents, edit only the `_*.md` file — all agents that reference it pick up the change automatically via the Read tool.

## Orchestrators must run as slash commands, not as agents

**The Agent tool is not available to subagents.** When Claude Code spawns a subagent, the subagent cannot itself spawn further subagents via the Agent tool — even if `tools: Agent` is listed in its frontmatter. This means `plan-manager` and `code-manager` cannot function as subagents: they would need to use the Agent tool to spawn plan-developer/reviewer/etc., but that tool is unavailable to them.

**Rule:** Orchestrators (`plan-manager`, `code-manager`) must run as **slash commands** in the main session, which does have the Agent tool. Their slash command files (`commands/plan-manager.md`, `commands/code-manager.md`) contain three steps: parse args → Read the `_*.md` instruction file → execute the pipeline. Never spawn plan-manager or code-manager as a subagent from another agent.

Leaf agents (`plan-developer`, `plan-reviewer`, `code-developer`, `code-reviewer`, `code-tester`, `test-reviewer`) are proper subagents — they only need Read/Write/Bash to do their work.
