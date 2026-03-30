---
name: commit-all-repos
description: Iterate through all sub-repos in repos/ and commit each one, then commit the master repo
user-invocable: true
---

# Commit All Repos

Iterate through every sub-repo in `repos/` and commit changes in each one individually, then commit the master repo last.

## Order of Operations

1. **Sub-repos first** — iterate through each directory in `repos/*/` that is a git repository
2. **Master repo last** — commit root-level changes after all sub-repos are done

## For Each Sub-Repo

For each repo under `repos/<name>/`:

1. `cd` into the repo directory
2. Run `git status` (never use `-uall`) — if no changes, skip it
3. Run `git diff` to review what changed
4. Run `git log --oneline -5` to match that repo's commit message style
5. **Read the sub-repo's agentic instructions** if they exist (in priority order):
   - `CLAUDE.md` in the sub-repo root
   - `AGENTS.md` in the sub-repo root
   - `.github/copilot-instructions.md` in the sub-repo
   - These instructions **take precedence** over root-level instructions for that repo's commit
6. Follow the sub-repo's commit message guidelines if they define any; otherwise fall back to the root `.claude/prompt-snippets/commit-message.md`
7. Stage relevant files (use explicit paths, not `git add -A`)
8. Create the commit
9. Run `git status` to verify

## For the Master Repo

After all sub-repos are committed:

1. Run the `/commit` skill for the root workspace (which excludes `repos/`)
2. If there are no root-level changes, skip — don't create an empty commit

## Notes

- Report a summary at the end: which repos had commits, which were skipped (clean), and any errors
- If a sub-repo has no changes, skip it silently — don't create empty commits
- If a pre-commit hook fails in a sub-repo, report the error and continue to the next repo
- Never force-push or amend existing commits in sub-repos
