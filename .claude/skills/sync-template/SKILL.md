---
name: sync-template
description: Pull latest repo-of-repos template updates into this workspace
user-invocable: true
---

# Sync Template

Pull the latest repo-of-repos template changes into this workspace without overwriting project-specific content.

## Overview

This workspace was created from the [repo-of-repos](https://github.com/raffertyuy/repo-of-repos) template. The template evolves — new skills, improved agents, better docs. This skill brings those updates into your workspace while preserving your project-specific configuration.

## Arguments

The user may optionally specify:
- `/sync-template` — use default upstream (`raffertyuy/repo-of-repos`, branch `main`)
- `/sync-template <github-org/repo>` — use a custom upstream fork
- `/sync-template --branch <branch>` — use a specific branch

## Order of Operations

1. **Check versions** — compare local vs upstream
2. **Fetch upstream** — clone template to a temp directory
3. **Read changelog** — understand what changed and why
4. **Copy safe files** — framework files that have no project-specific content
5. **Smart-merge mixed files** — files with both template and project content
6. **Update version** — write the new `TEMPLATE_VERSION`
7. **Report** — summarize what changed

## Step 1: Check Versions

1. Read the local `TEMPLATE_VERSION` file at the project root. If it doesn't exist, assume `0.0.0` (pre-versioned workspace).
2. Fetch only the upstream `TEMPLATE_VERSION` file:
   ```bash
   gh api repos/<upstream>/contents/TEMPLATE_VERSION --jq '.content' | base64 -d
   ```
3. Compare versions. If already up to date, inform the user and stop.
4. If the upstream is newer, continue.

## Step 2: Fetch Upstream

Clone the upstream template to a temporary directory:

```bash
git clone --depth 1 --branch <branch> https://github.com/<upstream>.git /tmp/repo-of-repos-upstream
```

Clean up this directory when done (in the final step).

## Step 3: Read Changelog

Read `CHANGELOG.md` from the upstream clone. Identify all versions between the local version and the upstream version. This tells you:
- **What files changed** and why
- **Migration notes** — special handling needed (e.g., `.gitignore` strategy change)
- **New files** that need to be created locally

Present a summary to the user before proceeding: which versions will be applied and what the key changes are.

## Step 4: Copy Safe Files

These are framework files with no project-specific content. Copy them directly from the upstream clone, overwriting local versions:

| Path | What |
|------|------|
| `.claude/skills/*/SKILL.md` | All skill definitions |
| `.claude/agents/*.md` | Agent definitions |
| `.claude/prompt-snippets/*.md` | Shared prompt snippets |
| `.claude/rules/*.md` | Auto-applied rules |
| `.github/agents/*.md` | Copilot agent mirrors |
| `.github/instructions/*.instructions.md` | Copilot instruction mirrors |
| `_tasks/README.md` | Task system docs |

**New files**: If the upstream has new skills, agents, rules, or prompt snippets that don't exist locally, create them (including any new directories).

**Deleted files**: If the upstream removed a skill or agent that exists locally, warn the user but do NOT delete — it may be a local addition.

## Step 5: Smart-Merge Mixed Files

These files contain both template content and project-specific content. You (the agent) are the merge tool. For each file:

1. Read the **upstream version** (from temp clone)
2. Read the **local version** (from workspace)
3. Read the relevant **changelog entries** for context on what changed and why
4. **Intelligently merge** — apply the template updates while preserving project-specific content

### CLAUDE.md

- Update template sections (Project Structure, Workspace Manifest, Git vs Local, Read/Write Separation, etc.)
- Preserve project-specific sections the user may have added (architecture notes, data flows, team conventions)
- Preserve any persona customizations

### README.md

- Update template sections (Getting Started, Skills, Agents, Key Concepts, etc.)
- Preserve project-specific sections (custom project tree, project description, etc.)
- Pay special attention to the project structure tree — it may have project-specific entries

### .gitignore

- Apply the template's ignore strategy (e.g., per-repo entries vs blanket glob)
- Preserve any project-specific ignore entries
- If the strategy changed (e.g., blanket `repos/*/` → per-repo entries), generate the correct entries based on the current `repos/repos.yaml`

### repos/repos.yaml

- Update **header comments only** (field documentation)
- **NEVER** touch actual repo entries — those are entirely project-specific

### CHANGELOG.md

- Copy directly from upstream (this file is template-owned)

## Step 6: Update Version

Write the upstream version string to the local `TEMPLATE_VERSION` file.

## Step 7: Cleanup and Report

1. Remove the temp clone directory: `rm -rf /tmp/repo-of-repos-upstream`
2. Report a summary:

```
## Template Sync: v0.2.0 → v0.3.0

### Files updated
| File | Action |
|------|--------|
| `.claude/skills/pull-all-repos/SKILL.md` | Copied from upstream |
| `CLAUDE.md` | Merged (added "Git vs Local Entries" section) |
| `.gitignore` | Merged (switched to per-repo entries) |
| ... | ... |

### New files
- `.claude/skills/sync-template/SKILL.md`

### Key changes
- Local source folder support (see CHANGELOG.md for details)

### Next steps
- Review the changes with `git diff`
- Commit when satisfied: `/commit`
```

## Notes

- **Never delete local files** unless they're template-owned and were removed upstream (and even then, warn first)
- **Never touch** `repos/repos.md`, `repos/*/`, `_tasks/*.md` (actual tasks), `.claude/settings*.json`, `.mcp.json`, `.vscode/mcp.json`
- **Always preserve** project-specific content in mixed files — when in doubt, keep both
- If the merge is ambiguous, show both versions to the user and ask which to keep
- The changelog is your guide — read it before merging to understand intent, not just diffs
