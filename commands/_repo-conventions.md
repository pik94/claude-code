You maintain the repository rules — per-domain convention files that pipeline agents (code-developer, code-reviewer, code-tester, test-reviewer) follow. The files live inside the project being worked on, under `./.claude/rules/` by default — not in `~/.claude`.

## Rule scopes

| File | Covers |
|---|---|
| `code.md` | Source code: naming, style, structure, error handling, where things live |
| `testing.md` | Tests: framework, runner command, directory layout, file naming, fixture/factory/mocking patterns, assertion style |
| `deploy.md` | Deploy artifacts: Dockerfiles, CI/CD configs, k8s manifests, release process |
| `tooling.md` | Package manager, task runner, how to run lint / tests / build |

Users may add their own scope files (e.g. `api.md`) — orchestrators route unrecognized scopes to code-developer.

## Arguments

Parse your input for these fields (apply defaults for any not provided):
- `mode` — `init` or `add` (default: `init`)
- `rule-text` — the rule to append (required for `add`)
- `scope` — which scope file an added rule goes to (default: `code`)
- `dir` — rules directory (default: `./.claude/rules`)

## Mode: init

1. **Explore.** Use the **`Agent` tool** (`subagent_type: Explore`) with:

   ```
   Explore this project's development conventions. Find: the test directory layout and file naming; the testing framework and its config; 3–5 representative test files; fixture/factory/mocking patterns; linter and formatter configs; language and tooling versions; the package manager and how tests/lint/build are run; deploy artifacts (Dockerfiles, CI/CD configs, k8s manifests). Thoroughness: very thorough.
   ```

2. **Verify by reading.** Read the representative files and configs that Explore located. Every rule you write must be backed by something you actually saw in the repository — do NOT invent rules or import generic best practices the repo does not follow.

3. **Write the scope files.** Create `{dir}` if needed. Write one file per scope, but ONLY for scopes with actual evidence — do not create `deploy.md` if the repo has no deploy artifacts. Each file:

   ```markdown
   # <Scope> conventions

   - <short, imperative, checkable rules>

   ## Manual rules
   - <rules added via /repo-conventions add — preserved verbatim, never auto-edited>
   ```

   Every rule must be short, imperative, and checkable (e.g. "Tests are plain pytest functions — never test classes", not "tests should follow good practices").

   If a scope file already exists, Read it first and update the generated rules in place — NEVER delete, reword, or reorder entries under `## Manual rules`.

   If a legacy `./.claude/conventions.md` exists, fold its rules into the matching scope files (its manual rules go into the matching file's `## Manual rules` section verbatim), then tell the user the legacy file is superseded and can be removed — do NOT delete it yourself.

4. **Report.** Show each created/updated file and note which repository evidence each generated rule set is based on.

## Mode: add

1. If `{dir}/{scope}.md` does not exist, create it (and `{dir}` if needed) with the structure above and empty generated section — no exploration needed.
2. Append `- {rule-text}` under `## Manual rules` of `{dir}/{scope}.md`.
3. Show the updated `## Manual rules` section.

## Rules
- In `init` mode, never write a rule without evidence from the repository
- Never modify or delete existing entries under `## Manual rules`
- Never create scope files for domains the repo has no evidence of
- This command only writes rules files — never touch source code or tests

$ARGUMENTS
