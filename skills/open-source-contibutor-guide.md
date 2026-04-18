---
name: opensource-contribution
description: >
  Use this skill whenever the user has a fix, feature, or improvement they want to contribute
  to an open source project on GitHub (or GitLab/Bitbucket). Triggers include: "we have a fix
  for a library", "how do I open a PR", "how do I report a bug upstream", "contribute to open
  source", "submit a patch", "open an issue on GitHub", "fork and fix", "upstream our changes".
  Always use this skill when the user mentions contributing code to someone else's repository,
  even if they just say "I want to send this fix to the library authors".
---

# Open Source Contribution Skill

Helps Claude Code users go from "we have a fix" to a merged upstream contribution — step by step.

## Input

Expect one or more of:
- Repository URL (GitHub / GitLab / Bitbucket)
- Description of the fix or feature
- The actual code diff or files

If any of these are missing, ask before proceeding.

---

## Step 0 — Recon: read the repo's own rules first

Before writing any instructions, fetch the repo to understand its specific requirements.

```bash
# Check for contribution guidelines
gh repo view <owner>/<repo> --json name,description,defaultBranchRef

# Look for key files
for f in CONTRIBUTING.md CONTRIBUTING.rst .github/CONTRIBUTING.md \
          CODE_OF_CONDUCT.md .github/PULL_REQUEST_TEMPLATE.md \
          .github/ISSUE_TEMPLATE; do
  gh api repos/<owner>/<repo>/contents/$f 2>/dev/null | jq -r '.content' | base64 -d
done
```

Key things to extract:
- Does the repo require an issue BEFORE a PR? (Many do — "open an issue first")
- Is there a CLA (Contributor License Agreement)?
- What branch to target? (`main`, `master`, `develop`, `next`?)
- Are there any required checks — tests, linting, changelog entry?
- Is there a PR template? (fill it out fully — maintainers notice when it's ignored)

Read `references/repo-patterns.md` for common patterns by ecosystem.

---

## Step 1 — Check if issue / PR already exists

```bash
# Search existing issues
gh issue list --repo <owner>/<repo> --search "<keywords from fix>" --state all

# Search PRs
gh pr list --repo <owner>/<repo> --search "<keywords>" --state all
```

**Decision tree:**
- Already merged → nothing to do, update your dependency
- Open PR exists → consider contributing to it instead of opening a duplicate
- Open issue exists, no PR → reference it in your PR (`Fixes #N`)
- Nothing exists → open an issue first (unless the fix is trivially small — typo, doc fix)

---

## Step 2 — Open an Issue (when needed)

**When to open an issue first:**
- Bug fix for non-obvious behaviour
- Any behavioural change, even if it's clearly a bug
- Security issues → check if the repo has a `SECURITY.md` and use private disclosure instead

**When you can skip the issue and go straight to PR:**
- Typo / formatting / docs only
- The CONTRIBUTING.md explicitly says "small fixes — PR directly"
- You already discussed it with a maintainer

**Issue content checklist:**
```
## What happened
Clear description of the bug / gap.

## Steps to reproduce
1. ...
2. ...

## Expected behaviour
...

## Actual behaviour
...

## Environment
- Library version:
- Language/runtime version:
- OS:

## Proposed fix
Optional — brief description, not the full diff yet.
```

```bash
gh issue create \
  --repo <owner>/<repo> \
  --title "<concise title: verb + noun, e.g. 'Fix memory leak in connection pool'>" \
  --body-file issue_body.md \
  --label bug   # or enhancement, documentation, etc.
```

---

## Step 3 — Fork & prepare branch

```bash
# Fork (--clone creates a local copy automatically)
gh repo fork <owner>/<repo> --clone --remote

cd <repo>

# Always branch from the repo's default/target branch
git checkout -b fix/<short-slug>   # e.g. fix/connection-pool-leak
# or
git checkout -b feat/<short-slug>
```

Branch naming conventions (adjust to what CONTRIBUTING.md says):
- `fix/` — bug fixes
- `feat/` — new features  
- `docs/` — documentation only
- `chore/` — tooling, CI, deps

---

## Step 4 — Apply the fix & verify

```bash
# Apply changes, then run the repo's own test suite
# Check CONTRIBUTING.md or Makefile for the right commands

# Common patterns:
make test          # C/C++/Go/Rust projects
npm test           # Node.js
pytest             # Python
cargo test         # Rust
go test ./...      # Go
bundle exec rspec  # Ruby
```

**Before committing:**
- [ ] All existing tests pass
- [ ] New tests added for the fix (if applicable)
- [ ] Linter / formatter passes (run `make lint`, `npm run lint`, `ruff check`, etc.)
- [ ] CHANGELOG updated (check if required — look for `CHANGELOG.md`, `CHANGES.rst`, or `NEWS`)
- [ ] No debug prints / commented-out code left behind

**Commit message:**
```
fix: <short description in imperative mood>

<Optional body: why the fix is needed, what the root cause was>

Fixes #<issue_number>
```

Follow Conventional Commits if the repo uses it (look for `.commitlintrc` or `commitlint.config.js`).

---

## Step 5 — Open the PR

```bash
git push origin fix/<short-slug>

gh pr create \
  --repo <owner>/<repo> \
  --title "fix: <same as commit message>" \
  --body-file pr_body.md \
  --base <target-branch>
```

**PR body template** (adapt to repo's own template if one exists):
```markdown
## Summary
Brief description of what the PR does.

## Related issue
Fixes #<issue_number>

## Changes
- What was changed and why

## How to test
Steps for the maintainer to verify the fix works.

## Checklist
- [ ] Tests pass
- [ ] New tests added
- [ ] Docs updated (if applicable)
- [ ] CHANGELOG updated (if required)
```

---

## Step 6 — After opening the PR

1. **Watch CI** — if checks fail, fix them promptly (maintainers ignore PRs with broken CI)
2. **Respond to reviews quickly** — slow responses kill PRs
3. **Keep the branch up to date:**
   ```bash
   git fetch upstream
   git rebase upstream/<target-branch>
   git push --force-with-lease origin fix/<short-slug>
   ```
4. **Don't force-push after approval** — some maintainers re-review from scratch if you do
5. **If no response after 2–4 weeks** — ping politely once in the issue/PR thread

---

## Special cases

See `references/repo-patterns.md` for ecosystem-specific patterns:
- Python (PyPI) — changelog format, tox, mypy
- Node.js (npm) — changesets, conventional commits
- Rust (crates.io) — cargo fmt, clippy, semver
- Go modules
- Security disclosures

---

## Quick-start summary

```
1. gh repo view → read CONTRIBUTING.md
2. gh issue/pr list → check for duplicates
3. gh issue create → (if needed)
4. gh repo fork --clone
5. git checkout -b fix/<slug>
6. [apply fix + tests]
7. git push → gh pr create
```