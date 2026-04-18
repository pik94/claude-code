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
plan-developer-{tier}    — writes structured implementation plans (no code changes)
plan-reviewer-{tier}     — critiques plans, returns structured feedback (no code changes)

plan-manager             — orchestrates: plan-developer ⇄ plan-reviewer × N → plan_final.md
code-manager             — orchestrates: code-developer → code-reviewer (fix if critical) → code-tester
plan-and-code-manager    — orchestrates: plan-manager → code-manager end-to-end
```

**Explore agent:** `code-developer`, `code-reviewer`, and `plan-developer` agents delegate broad codebase searches to `Explore` (Haiku) instead of running Glob/Grep themselves. Pass a description plus thoroughness level (`quick` / `medium` / `very thorough`). Use Read directly only for known paths.

**Key invariant:** agents share no memory. Orchestrators must pass the full text of prior outputs when invoking the next agent — never rely on return text alone; read saved files from disk.

**Handoff convention:** `plan-manager` writes its canonical output to `{plan-dir}/plan_final.md`. `plan-and-code-manager` reads this file from disk before invoking `code-manager`.

**Fix loop:** `code-manager` only triggers a `code-developer` fix pass when the reviewer flags **critical** issues (crashes, null dereferences, data loss, security). Warnings and style issues do not trigger a fix pass.

## Slash commands

Commands without a `_` prefix are user-facing (become `/command-name`). Files with a `_` prefix are instruction files loaded by agents — not directly invokable.

| Command | Delegates to | Key flags |
|---|---|---|
| `/code-developer` | `code-developer-medium` | `--agent=code-developer-{low,medium,high}` |
| `/code-reviewer` | `code-reviewer-medium` | `--agent=code-reviewer-{low,medium,high}` |
| `/code-tester` | `code-tester-medium` | `--agent=code-tester-{low,medium,high}` |
| `/plan-developer` | `plan-developer-medium` | `--agent=plan-developer-{low,medium,high}` |
| `/plan-reviewer` | `plan-reviewer-medium` | `--agent=plan-reviewer-{low,medium,high}` |
| `/code-manager` | `code-manager` | `--code-developer=`, `--code-reviewer=`, `--code-tester=` |
| `/plan-manager` | `plan-manager` | `--plan-developer=`, `--plan-reviewer=`, `--plan-dir=`, `--plan-iterations=` |
| `/plan-and-code-manager` | `plan-and-code-manager` | all of the above; `--plan-dir` defaults to `./plans` |

## Settings

`settings.json` sets `defaultMode: auto` and pre-allows common tools (git, uv, make, docker, python3, pip, npm, node, ls, cat, find, grep, Agent) so they never prompt. Destructive commands (`rm`, `docker rm`, `git reset`) require confirmation.

## Conventions when adding agents or commands

- Agent frontmatter must include `name`, `description`, and `tools` (restrict to minimum needed). Add `model` only when the tier-based naming doesn't apply.
- Planner-type agents must state in their rules: they may not modify source code, only write markdown files.
- User-facing command files have a YAML frontmatter block with `description` and `argument-hint`. Instruction files (prefixed `_`) have no frontmatter — they are raw prompt text.
- Orchestrators resolve all optional args to their defaults at the very top before invoking any subagent.
- Output directories for each pipeline stage must be distinct (never share `plan-dir` and dev output dir).
- Pipeline output files follow the pattern: `{dir}/plan_v{n}.md`, `{dir}/review_v{n}.md`, `{dir}/plan_final.md`.
