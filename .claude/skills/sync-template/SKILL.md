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

Read `TEMPLATE_CHANGELOG.md` from the upstream clone. Identify all versions between the local version and the upstream version. This tells you:
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

CLAUDE.md is the most complex merge target. Live projects customize it heavily — different persona, project-specific architecture, custom coding standards, additional sections. The template evolves its framework sections independently.

**Classify every section as one of:**

| Classification | What it means | Merge strategy |
|----------------|---------------|----------------|
| **Template-owned** | Section exists in both template and local, content is purely framework | Replace with upstream version |
| **Template-seeded, project-customized** | Section originated from template but the project has modified it | Three-way merge (see below) |
| **Project-only** | Section exists only in the local file | Preserve as-is, keep in its current position |

**Template-owned sections** (replace with upstream):

- Project Structure (intro paragraph, bullet list of git repos / local folders)
- Per-Repo Instructions (table of `CLAUDE.md` / `AGENTS.md` / `copilot-instructions.md`)
- Workspace Manifest (the `repos.yaml` description paragraph)
- Git vs Local Entries (comparison table)
- Read/Write Separation
- Task System
- Prompt Snippets (the explanation paragraph)
- Agentic Configuration Sync (table + rules)
- Self-Improvement

**Template-seeded sections that projects commonly customize** (three-way merge):

- **Persona / Voice & Personality** — The template ships a default persona (Tony Stark). Projects may keep it, modify it, or replace it entirely. If the local version differs from the template default, **preserve the local persona**. Only apply upstream changes if the template's persona section gained new structural elements (e.g., new `Do NOT` rules) — append those to the local version.
- **Writing Style** — Projects may add style rules. Preserve local additions, update template-originated rules.
- **Coding Standards** — The template references a prompt snippet. Projects may add inline standards. Preserve local additions.
- **Commit Message Style** — Same as above.

**Three-way merge procedure:**

1. Read the **upstream** section
2. Read the **local** section
3. Identify what the local version **added, removed, or changed** relative to what the template would have had
4. Apply the upstream update as the new base
5. Re-apply the local modifications on top
6. If a local modification conflicts with an upstream change (both changed the same content), **prefer the local version** and add a comment: `<!-- sync-template: upstream also changed this section — review manually -->`

**Project-only sections** — Anything in the local CLAUDE.md that has no counterpart in the upstream template. Common examples:

- Architecture notes, system diagrams
- Data flow descriptions
- API contracts, schema references
- Team conventions, on-call notes
- Environment-specific setup
- Custom rules or overrides

These must be **preserved in their current position** in the file. If an upstream update adds a new template section, insert it in the correct position relative to other template sections, but never displace project-only sections.

**Section ordering:** Follow the upstream's section order for template sections. Project-only sections stay where the user placed them. If a project-only section is between two template sections, keep it between them (or at the end if the surrounding template sections were reordered).

**First-line / title:** The local CLAUDE.md may have a different title or opening line (e.g., `# aws-mk0` instead of `# CLAUDE.md`). Preserve the local title.

### README.md

README.md is the public face of the project. Live projects replace the template description, customize the project tree, add project-specific sections, and may remove template sections that don't apply.

**Classify every section as one of:**

| Classification | What it means | Merge strategy |
|----------------|---------------|----------------|
| **Template-owned** | Pure framework documentation | Replace with upstream version |
| **Template-seeded, project-customized** | Originated from template, modified by project | Three-way merge |
| **Project-only** | Exists only in the local file | Preserve as-is |
| **Template-removed** | Exists in template but project intentionally deleted it | Do NOT re-add |

**Template-owned sections** (replace with upstream):

- Key Concepts > Read/Write Separation
- Key Concepts > Task System
- Key Concepts > Workspace Manifest (repos.yaml)
- Cross-Repo PR Linking
- Cross-Tool Sync (Claude Code + GitHub Copilot)
- Customization
- Prior Art & Inspiration
- License

**Template-seeded sections that projects commonly customize** (three-way merge):

- **Project title and description** — The very top of the README. Projects replace the template name, description, and "The Problem / The Solution" with their own. **Always preserve the local version.** If the upstream added an "upstream template link" line and the local doesn't have it, suggest adding it but don't force it.
- **Project Structure (tree)** — Projects add their own repos, remove example entries, add non-repo directories. Preserve the local tree structure. Only update the **template framework entries** (`.claude/`, `.github/`, `_tasks/`, etc.) if the upstream added new files or directories to those paths. Never remove project-specific tree entries.
- **Getting Started** — Projects may simplify or customize setup steps. Preserve local modifications. Update only if the upstream changed core workflow (e.g., new required step, changed command syntax).
- **Skills (Slash Commands)** — The table of available skills. **Regenerate from upstream** but preserve any project-specific skills the local version added. Check: if the local table has a skill not in the upstream, keep it. If upstream added a new skill, add it.
- **Agents** — Same strategy as Skills — merge tables, preserve local additions.
- **MCP Servers** — Preserve the local table (projects add their own servers). Only add new entries from upstream if the template introduced a new default MCP server.
- **Keeping Up to Date** — Template-owned, replace with upstream.

**Detecting intentionally removed sections:**

If a template section exists in the upstream but NOT in the local README, the project likely removed it intentionally. **Do NOT re-add it.** Instead, note it in the sync report:

```
| Key Concepts > Workspace Manifest | Skipped (not present locally — likely intentionally removed) |
```

If unsure whether removal was intentional, ask the user before re-adding.

**Three-way merge procedure:** Same as CLAUDE.md (see above).

**Section ordering:** Follow the local file's ordering. If upstream introduces a new section, insert it in a logical position relative to existing sections.

### .gitignore

- Apply the template's ignore strategy (e.g., per-repo entries vs blanket glob)
- Preserve any project-specific ignore entries
- If the strategy changed (e.g., blanket `repos/*/` → per-repo entries), generate the correct entries based on the current `repos/repos.yaml`

### repos/repos.yaml

- Update **header comments only** (field documentation)
- **NEVER** touch actual repo entries — those are entirely project-specific

### TEMPLATE_CHANGELOG.md

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
- Local source folder support (see TEMPLATE_CHANGELOG.md for details)

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
