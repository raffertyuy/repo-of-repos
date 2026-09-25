# Changelog

All notable changes to the repo-of-repos template. Run `/sync-template` to pull the latest into your workspace.

## 0.6.1

### Fix agents not loading in Claude Code, and frontend rule never matching

- **Agents**: added `description` to `explorer`, `worker`, and `reviewer` frontmatter. Claude Code skips agents without one, so none of the three were registering — the orchestrator fell back to general-purpose agents.
- **`reviewer`**: dropped `Edit` from its tools to match its read-only role. Review standards are now inlined in `reviewer.md`.
- **`.claude/prompt-snippets/review-standards.md` removed**: it had only one consumer (`reviewer.md`), which breaks the 2+ consumer rule for snippets.
- **`.claude/rules/frontend.md`**: `paths` globs changed from `src/**/*` to `**/src/**/*`. The old globs only matched a root-level `src/`, never `repos/<name>/src/`.
- **Docs**: `docs/cross-tool-sync.md` now notes that agents need `description` and that rule globs need a `**/` prefix. README agents table notes that `explorer` has `Bash` and is read-only by instruction.

**Migration note**: run `/sync-template`. Then delete `.claude/prompt-snippets/review-standards.md` — it has no `origin: template` marker, so removed-file detection won't flag it. If you customized it, move your changes into the `## Review Standards` section of `.claude/agents/reviewer.md`, then re-apply after future syncs (agents are overwritten on sync).

## 0.6.0

### Cross-tool compatibility: Claude Code + GitHub Copilot + OpenAI Codex

Restructured agentic config so all three tools read the same canonical files, per [Claude + Copilot + Codex Cross-Compatibility](https://raffertyuy.com/raztype/claude-copilot-codex-cross-compatibility/). One canonical location per feature — no duplicated content.

- **`AGENTS.md`** (new): canonical instruction file, read natively by Copilot and Codex. All content moved here from `CLAUDE.md`.
- **`CLAUDE.md`**: reduced to a one-line pointer — `@AGENTS.md`. Claude Code expands the import.
- **Skills moved to `.agents/skills/`**: canonical `SKILL.md` files now live in `.agents/skills/<name>/` (read natively by Copilot and Codex). `.claude/skills/<name>/SKILL.md` are now thin stubs — duplicated frontmatter plus `@../../../.agents/skills/<name>/SKILL.md` (Claude Code reads only the stub's frontmatter for discovery).
- **`.codex/config.toml`** (new): Codex MCP server mirror of `.mcp.json` (`[mcp_servers.name]` TOML sections). Keep both in sync.
- **`.github/agents/` removed**: VS Code and Copilot CLI now read Claude-format agents from `.claude/agents/` directly — the mirrors were stale duplicates.
- **`.github/instructions/` removed**: VS Code Copilot now reads `.claude/rules/`. The frontend standards content moved into `.claude/rules/frontend.md` (previously it only pointed at the Copilot file).
- **`.vscode/mcp.json` removed**: VS Code 1.118+ reads the root `.mcp.json`. Older VS Code users can recreate `.vscode/mcp.json` with a `"servers"` key if needed.
- **`/sync-template`**: safe-copy list, removed-file detection, and merge instructions updated for the new layout. The `CLAUDE.md` merge section is now the `AGENTS.md` merge section, with a pre-0.6.0 migration path.
- **`/update-all-md-docs`**: scope and consistency checks updated (`.agents/`, `.codex/`, skill-stub validation, `.codex/config.toml` MCP table check).
- **Docs**: `docs/cross-tool-sync.md` rewritten for three tools. `README.md` structure tree, skills/agents/MCP/customization sections updated.

**Migration note**: run `/sync-template`. It will: (1) create `AGENTS.md` from your customized `CLAUDE.md` content and replace `CLAUDE.md` with the one-line pointer, (2) move skills to `.agents/skills/` and create stubs in `.claude/skills/`, (3) create `.codex/config.toml` mirroring your `.mcp.json` servers, and (4) prompt to delete `.github/agents/`, `.github/instructions/` (after merging any customized rule content into `.claude/rules/`), and `.vscode/mcp.json` (keep it if your VS Code is older than 1.118).

## 0.5.6

### Show nested git sub-repos in VS Code Source Control

- **`.vscode/settings.json`**: added `"git.repositoryScanMaxDepth": 3`. VS Code's git scanner defaults to depth `1`, which doesn't reach `repos/<name>/.git` (depth 2). With this raised to 3, every git-type sub-repo in `repos/` shows up as its own entry in the Source Control panel — each with its own branch indicator, staging area, and Commit button. Local-type entries (no `.git`) continue to be tracked by the parent repo as before.

## 0.5.5

### Rename plan skills to avoid Copilot collision

GitHub Copilot ships a default `/plan` command, making the template's `/plan` skill uncallable in Copilot. Renamed both plan skills to avoid the collision.

- **`/plan` → `/create-plan`**: skill directory renamed (`.claude/skills/plan/` → `.claude/skills/create-plan/`). SKILL.md `name` field and examples updated.
- **`/implement` → `/implement-plan`**: skill directory renamed (`.claude/skills/implement/` → `.claude/skills/implement-plan/`). SKILL.md `name` field and examples updated. Renamed for symmetry with `/create-plan`.
- **Docs**: updated references in `CLAUDE.md`, `README.md`, and `_plans/README.md` to use the new command names.

**Migration note**: Live projects should run `/sync-template` to pick up the renames. The `/sync-template` skill will detect the old `.claude/skills/plan/` and `.claude/skills/implement/` directories as removed template files (via `origin: template` frontmatter) and prompt for deletion. Confirm to remove the old directories. New skills appear at `.claude/skills/create-plan/` and `.claude/skills/implement-plan/`.

## 0.5.4

### Expanded search.exclude for multi-language repos

- **`.vscode/settings.json`**: added broad `search.exclude` patterns covering JS/TS (`dist/`, `.next/`, `.nuxt/`), Python (`.venv/`, `venv/`, `__pycache__/`, `.pytest_cache/`, `.mypy_cache/`, `.ruff_cache/`), .NET (`bin/`, `obj/`), Java/Kotlin (`target/`, `.gradle/`), Go (`vendor/`), IaC (`.pulumi/`), and general artifacts (`build/`, `out/`, `coverage/`, `.cache/`). These prevent build outputs and dependency caches from polluting workspace search results across heterogeneous repos.

## 0.5.3

### Fix /pull-all-repos orphan detection on empty manifest

- **`/pull-all-repos`**: fixed early exit when `repos.yaml` has an empty list (`repos: []`). Previously, an empty manifest would short-circuit with a "nothing to do" message, skipping orphan detection entirely. Now skips steps 2–4 and proceeds directly to step 5 (orphan detection), so manually added folders in `repos/` are always discovered and registered.

## 0.5.2

### commit/commit-all-repos now stage and push; sync-template detects removed template files

- **`/commit`**: now does `git add` (explicit paths), `git commit`, and `git push` in one flow. No more committing only what was manually staged.
- **`/commit-all-repos`**: same — each sub-repo is staged, committed, and pushed. Description updated to match.
- **`origin: template` frontmatter**: added `origin: template` field to all template-owned skill SKILL.md files and agent files. This is the canonical marker that distinguishes template files from project-custom ones.
- **`/sync-template` Step 4**: new "Removed template files" logic — scans local skills/agents with `origin: template`, checks each against upstream, and prompts the user to delete any that no longer exist in the template. Skills/agents without `origin: template` are never touched.
- **`/sync-template` report**: sync summary now includes a "Removed files" section.
- **`/sync-template` notes**: documents the `origin: template` contract for future skill authors.

**Migration note**: Live projects with `/create-task` and `/list-tasks` skills should manually delete those directories (`.claude/skills/create-task/` and `.claude/skills/list-tasks/`). Future syncs will detect and prompt removal automatically.

## 0.5.1

### Simpler plan filenames

- **Plan naming**: changed from `<prefix>-<number>-<slug>.plan.md` to `YYYYMMDD-<plan-name>.plan.md`. Date prefix replaces sequential numbering and repo prefix routing.
- **`/plan`**: updated to use new naming format. Removed "Determine Scope" prefix logic and "Assign a Number" step.
- **`/implement`**: updated example filenames.
- **`_plans/README.md`**: updated naming format and examples.
- **`repos/repos.yaml`**: `prefix` field description simplified (no longer tied to plan routing).
- **Docs and references**: updated across `CLAUDE.md`, `README.md`, `docs/workspace-manifest.md`, and skill files.

**Migration note**: Existing plans with the old naming format (`fe-1-*.plan.md`) still work — `/implement` resolves by matching the plan name, not the prefix. No rename needed.

## 0.5.0

### Plan-implement workflow, README restructure, docs/ folder

Replaced the task system (`_tasks/`, `/create-task`, `/list-tasks`) with a two-phase plan-implement workflow inspired by the [Plan-Implement-Run pattern](https://raffertyuy.com/raztype/vibe-coding-plan-implement-run/).

- **`/plan`**: new skill — creates implementation plans in `_plans/` with steps, repo context, and pseudocode. Uses markdown task lists (`- [ ]`) for trackable progress.
- **`/implement`**: new skill — executes a plan, checks off steps (`- [x]`), runs tests, documents fixes and discoveries. Status auto-flows `draft` → `in-progress` → `completed`.
- **`_plans/`**: replaces `_tasks/`. Plans are living documents that record what was planned AND what actually happened.
- **`_plans/README.md`**: plan file format, status lifecycle, and how `/implement` updates plans.
- **`docs/`**: new folder — extracted detailed reference material from README to keep it lean.
  - `docs/adding-repos.md` — all options for adding git repos and local folders
  - `docs/workspace-manifest.md` — full `repos.yaml` field reference
  - `docs/cross-tool-sync.md` — Claude Code + Copilot sync rules
- **`README.md`**: restructured for junior developers. New flow: What → Setup → Use → Explore. Plan-implement workflow is now the featured usage section with walkthrough examples. Detailed reference material moved to `docs/`.
- **`CLAUDE.md`**: "Task System" section replaced with "Plan System".
- **`.claude/agents/worker.md`**: references `_plans/` instead of `_tasks/`.
- **`.claude/skills/sync-template/SKILL.md`**: updated section lists to match new README structure. Added `docs/*.md` to safe-copy files.
- **`.claude/skills/pull-all-repos/SKILL.md`**: prefix comment updated.
- **`repos/repos.yaml`**: header comment updated.

**Removed**: `/create-task`, `/list-tasks` skills and `_tasks/` directory.

**Migration note**: Delete `_tasks/` and its contents (or rename any active tasks to the new `_plans/` format manually). The new plan format uses `status: draft | in-progress | completed` in frontmatter and `- [ ]` / `- [x]` checkboxes for step tracking.

## 0.4.0

### Optional gitignore for local folders, versioning rules

- **`repos/repos.yaml`**: new `gitignore` field for `type: local` entries. `true` to gitignore, `false`/omitted to track (default). Git repos are always gitignored.
- **`/add-repository`**: now asks whether local folders should be tracked by git or gitignored.
- **`/pull-all-repos`**: orphan detection asks the same question for local orphans. Gitignore sync respects the `gitignore` field.
- **`/remove-repository`**: cleans `.gitignore` for any removed entry (git or gitignored local), not just git repos.
- **`/clean-repos`**: same — removes `.gitignore` lines for any stale entry type.
- **`CHANGELOG.md`**: renamed to `TEMPLATE_CHANGELOG.md` to avoid confusion in live projects.
- **`README.md`**: added `/sync-template` to skills table. Updated local folder docs to reflect optional gitignore. Added `gitignore` field to manifest example.

**Migration note**: Existing local folders default to tracked (no `gitignore` field = tracked). No action needed unless you want to gitignore a local folder — add `gitignore: true` to its entry in `repos/repos.yaml`.

## 0.3.0

### Local source folder support

`repos/` now supports both git repos and local (non-git) source folders.

- **`repos/repos.yaml`**: new `type` field (`git` | `local`). `url` is now optional for `type: local` entries.
- **`.gitignore`**: changed from blanket `repos/*/` to per-git-repo entries. Local folders are tracked by the root repo. When syncing, generate `.gitignore` entries for each git repo in your `repos.yaml`.
- **`/pull-all-repos`**: auto-detects and registers orphan directories (checks for `.git/` to determine type). Syncs `.gitignore` per type. Creates missing local folders.
- **`/add-repository`**: supports `--local` flag to create/register local source folders.
- **`/commit-all-repos`**: explicitly skips `type: local` entries (committed via root `/commit`).
- **`/create-task`**: scopes tasks to local folders same as git repos.
- **`CLAUDE.md`**: new "Git vs Local Entries" comparison table.
- **`README.md`**: new "Adding local source folders" section in Getting Started.
- **`_tasks/README.md`**: prefix routing updated to include local folders.

**Migration note**: Your `.gitignore` needs to switch from `repos/*/` to explicit entries per git repo. The `/sync-template` skill handles this automatically.

## 0.2.0

### Read/write separation, task system, workspace manifest, cross-repo PR linking

- **Agents**: `explorer.md` (read-only cross-repo), `worker.md` (write-scoped single-repo), `reviewer.md` (code review)
- **Skills**: `/pull-all-repos`, `/add-repository`, `/commit`, `/commit-all-repos`, `/pr-all-repos`, `/create-task`, `/list-tasks`, `/update-all-md-docs`
- **Task system**: `_tasks/` with prefix routing and context distillation
- **Workspace manifest**: `repos/repos.yaml` as declarative source of truth
- **Cross-repo PR linking**: `/pr-all-repos` creates and cross-links sibling PRs
- **Prompt snippets**: `coding-standards.md`, `commit-message.md`, `review-standards.md`
- **Rules**: `frontend.md` auto-applied rule
- **Cross-tool sync**: Claude Code + GitHub Copilot parity

## 0.1.0

### Initial scaffolding

- Project structure with `repos/`, `_tasks/`, `.claude/`, `.github/`, `.vscode/`
- MCP server config (Context7, Playwright) in both `.mcp.json` and `.vscode/mcp.json`
- Tony Stark persona in `CLAUDE.md`
- MIT license
