# Changelog

All notable changes to the repo-of-repos template. Run `/sync-template` to pull the latest into your workspace.

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
