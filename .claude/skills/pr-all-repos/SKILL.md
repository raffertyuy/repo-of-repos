---
name: pr-all-repos
description: Create PRs in all sub-repos with uncommitted branches, then link them as siblings
user-invocable: true
---

# PR All Repos

Create pull requests across all sub-repos that have branches ready for review, then cross-link them as sibling PRs.

## Prerequisites

- `gh` CLI must be authenticated (`gh auth status`)
- Each sub-repo must have a remote configured

## Order of Operations

1. **Scan** — identify repos with PR-ready branches
2. **Create PRs** — open a PR in each qualifying repo
3. **Cross-link** — update each PR description with sibling PR URLs

## Step 1: Scan

For each directory in `repos/*/` that is a git repository:

1. Check the current branch: `git -C repos/<name> branch --show-current`
2. **Skip** if on the default branch (main/master) — nothing to PR
3. Check if a PR already exists for this branch: `gh pr view --repo <remote> --json url 2>/dev/null`
4. **Skip** if a PR already exists — but record its URL for cross-linking
5. Check if the branch has been pushed: `git -C repos/<name> log @{u}..HEAD 2>/dev/null`
6. If unpushed commits exist, push first: `git -C repos/<name> push -u origin <branch>`

Collect the list of repos that need new PRs and repos that already have PRs.

## Step 2: Create PRs

For each repo that needs a new PR:

1. `cd` into the repo directory
2. Read `git log --oneline main..<branch>` (or master) to understand the changes
3. Draft a PR title (short, under 70 characters) and body
4. Create the PR:

```bash
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
<bullet points describing changes>

## Sibling PRs
<!-- Will be updated with cross-links -->

## Test Plan
- [ ] <testing steps>

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

5. Record the PR URL

## Step 3: Cross-Link

After all PRs are created (or found existing), do a second pass:

For each PR (new and existing), update its description to include a "Sibling PRs" section listing all other PRs in this batch:

```markdown
## Sibling PRs
- [frontend#42](https://github.com/org/frontend/pull/42) — Update auth UI components
- [infra#18](https://github.com/org/infra/pull/18) — Add new IAM role for auth service
```

Use `gh pr edit <number> --repo <remote> --body "<updated body>"` to update descriptions.

## Branch Name Convention

If all repos share the same branch name (e.g., `feature/ENG-123-auth-redesign`), note the ticket ID in the summary. If branch names differ across repos, still link them — the sibling PR section is the coordination mechanism.

## Arguments

The user may optionally specify:
- A base branch: `/pr-all-repos --base develop` (default: `main`)
- A subset of repos: `/pr-all-repos frontend backend-api` (default: all repos with non-default branches)

## Summary

Report a table at the end:

| Repo | PR | Status |
|------|-----|--------|
| `frontend` | [#42](url) | Created |
| `backend-api` | [#15](url) | Already existed — updated with sibling links |
| `infra` | — | Skipped (on main) |

## Notes

- Never force-push
- If `gh` is not authenticated, inform the user and stop
- If a repo has no remote, skip it with a warning
- Preserve any existing PR description content when adding sibling links — only add/update the "Sibling PRs" section
