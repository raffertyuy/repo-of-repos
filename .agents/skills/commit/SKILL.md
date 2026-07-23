---
name: commit
description: Stage, commit, and push changes for the master repo only (excludes sub-repos in repos/)
user-invocable: true
origin: template
---

# Commit (Master Repo Only)

Stage, commit, and push changes in the **root workspace only**. This skill does NOT touch any sub-repos under `repos/`.

## Scope

- Only stage and commit files in the root workspace
- **Exclude** everything under `repos/*/` — do not stage, commit, or modify anything in sub-repos
- If `git status` shows changes only in `repos/`, inform the user there's nothing to commit in the master repo and suggest using `/commit-all-repos` instead

## Instructions

1. Run `git status` (never use `-uall`) and `git diff` to review changes — **ignore any changes under `repos/`**
2. Run `git log --oneline -5` to match recent commit message style
3. Follow the commit message guidelines in `.claude/prompt-snippets/commit-message.md`
4. Stage only root-level files using explicit paths — never `git add -A` or `git add .` which could pull in sub-repo changes
5. Create the commit
6. Run `git push` to push the commit to the remote
7. Run `git status` to verify

If there are no root-level changes to commit, say so and stop.
