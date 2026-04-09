# repo-of-repos

A starter template for working with **multiple git repos and local source folders as a single AI-powered workspace**.

## The Problem

Real projects span many repos. AI tools work best when they see everything in one place. You don't want a monorepo.

## The Solution

Clone repos into `repos/`. Put local source code there too. Root-level agentic configs provide cross-repo context. Git repos stay independent (own git history, CI, deploys). Local folders are tracked by the root repo. Your AI gets the full picture.

This pattern goes by many names — "Virtual Monorepo," "Spine Pattern," "Polyrepo Synthesis." This template packages the best ideas into a ready-to-use workspace.

## Project Structure

```
.
├── CLAUDE.md                          # Root instructions for Claude Code
├── TEMPLATE_VERSION                   # Current template version
├── CHANGELOG.md                       # Template changelog
├── .mcp.json                          # MCP servers (Claude Code)
├── .vscode/
│   └── mcp.json                       # MCP servers (VS Code / Copilot)
├── .claude/
│   ├── settings.json                  # Claude Code settings
│   ├── agents/
│   │   ├── explorer.md                # Read-only cross-repo agent
│   │   ├── worker.md                  # Write-scoped single-repo agent
│   │   └── reviewer.md               # Code review agent
│   ├── rules/
│   │   └── frontend.md               # Auto-applied frontend rules
│   ├── skills/                        # Slash commands (see below)
│   │   ├── add-repository/
│   │   ├── commit/
│   │   ├── commit-all-repos/
│   │   ├── create-task/
│   │   ├── list-tasks/
│   │   ├── pr-all-repos/
│   │   ├── pull-all-repos/
│   │   ├── sync-template/
│   │   └── update-all-md-docs/
│   └── prompt-snippets/               # Shared instructions
│       ├── coding-standards.md
│       ├── commit-message.md
│       └── review-standards.md
├── .github/
│   ├── agents/                        # Copilot equivalents
│   │   ├── explorer.md
│   │   ├── worker.md
│   │   └── reviewer.md
│   └── instructions/
│       └── frontend.instructions.md
├── _tasks/                            # Task files with prefix routing
│   └── README.md
├── repos/
│   ├── repos.yaml                     # Workspace manifest — declares all entries
│   ├── repos.md                       # Auto-generated descriptions
│   ├── frontend/                      # (git repo — cloned, gitignored)
│   ├── backend-api/                   # (git repo — cloned, gitignored)
│   ├── infra/                         # (git repo — cloned, gitignored)
│   └── shared-config/                 # (local folder — tracked by root repo)
└── .gitignore
```

## Getting Started

### 1. Use This Template

Click "Use this template" on GitHub, or clone directly.

### 2. Add Your Code

The `repos/` directory supports two types of entries:

| | Git repos | Local folders |
|---|---|---|
| **Has own `.git`** | Yes | No |
| **Tracked by root repo** | No (gitignored) | Yes |
| **Cloned/pulled by `/pull-all-repos`** | Yes | Verified/created |
| **Committed by `/commit-all-repos`** | Per sub-repo | Via root `/commit` |
| **PRs via `/pr-all-repos`** | Yes | Skipped |

#### Adding git repos

**Option A: Manifest** (recommended)

Edit `repos/repos.yaml`:

```yaml
repos:
  - name: frontend
    url: git@github.com:your-org/frontend.git
    branch: main
    prefix: fe
    description: React frontend application

  - name: backend-api
    url: git@github.com:your-org/backend-api.git
    branch: main
    prefix: be
    description: REST API service
```

Run `/pull-all-repos` to clone everything.

**Option B: One at a time**

`/add-repository <url>` — clones, analyzes, updates `repos/repos.yaml` and `repos/repos.md`.

#### Adding local source folders

For source code that doesn't belong to any git repo — scripts, shared config, scratch projects, prototypes, etc.

**Option A: Slash command**

```
/add-repository --local shared-config
```

This will:
1. Create `repos/shared-config/` if it doesn't exist
2. Add a `type: local` entry to `repos/repos.yaml`
3. Update `repos/repos.md` with a description
4. Ensure the folder is NOT in `.gitignore` (so the root repo tracks it)

Then add your source files into `repos/shared-config/`.

**Option B: Manifest**

Add a `type: local` entry to `repos/repos.yaml`:

```yaml
repos:
  - name: shared-config
    type: local
    prefix: cfg
    description: Shared configuration and scripts
```

Run `/pull-all-repos` — it will create the directory if missing and register it.

Then add your source files into `repos/shared-config/`.

**Option C: Manual**

1. Create the folder: `mkdir repos/my-project`
2. Add your source files
3. Add an entry to `repos/repos.yaml` with `type: local`
4. Run `/pull-all-repos` to regenerate `repos/repos.md`

> **How it works**: Git repos get explicit `.gitignore` entries so the root repo ignores them. Local folders have no such entry, so they're tracked and committed with the root repo via `/commit`.

### 3. Customize

- `CLAUDE.md` — add project context, architecture notes, data flows
- `.claude/rules/` — add rules for your languages/frameworks
- `.claude/prompt-snippets/` — tune shared instructions

### 4. Start Working

Use slash commands below. The AI sees all repos and how they relate.

## Keeping Up to Date

This template evolves — new skills, improved agents, better docs. To pull the latest changes into your workspace:

```
/sync-template
```

This will:
1. Check your `TEMPLATE_VERSION` against the upstream
2. Copy updated framework files (skills, agents, prompt snippets, rules)
3. Smart-merge files that mix template and project content (`CLAUDE.md`, `README.md`, `.gitignore`)
4. Never touch your repos, tasks, settings, or MCP config

Check `CHANGELOG.md` to see what's new in each version. The skill reads it too — so it knows *why* things changed, not just *what* changed, and can merge intelligently.

## Skills (Slash Commands)

Invoked in Claude Code with `/<name>`. Defined in `.claude/skills/`.

### Workspace

| Command | What it does |
|---------|-------------|
| `/pull-all-repos` | Clone/pull git repos, verify local folders, auto-register orphans, regenerate `repos.md` |
| `/add-repository <url>` | Clone a git repo into `repos/` and register it |
| `/add-repository --local <name>` | Create/register a local source folder in `repos/` |

### Git

| Command | What it does |
|---------|-------------|
| `/commit` | Commit root workspace only (excludes `repos/`) |
| `/commit-all-repos` | Commit each sub-repo, then root workspace |
| `/pr-all-repos` | Create PRs in all sub-repos, cross-link as siblings |

### Tasks

| Command | What it does |
|---------|-------------|
| `/create-task <description>` | Create task file with prefix routing and auto-gathered context |
| `/list-tasks [filter]` | List tasks by status, prefix, or repo |

### Docs

| Command | What it does |
|---------|-------------|
| `/update-all-md-docs` | Sync all markdown files with current state |

### Template

| Command | What it does |
|---------|-------------|
| `/sync-template` | Pull latest template updates from upstream without overwriting project-specific content |

## Agents

Defined in `.claude/agents/` and mirrored in `.github/agents/` for Copilot.

| Agent | Scope | Purpose |
|-------|-------|---------|
| **explorer** | Read-only, all repos | Trace dependencies, find types, answer "what breaks if I change X?" |
| **worker** | Write-scoped, one repo | Implement changes. Follows target repo's own instructions. |
| **reviewer** | Read-only | Code review per `review-standards.md`. Runs on Sonnet. |

## Key Concepts

### Read/Write Separation

> Exploration is cross-repo. Writes are single-repo scoped.

- **Explorer** reads everything, writes nothing
- **Worker** writes to one repo, reads everything
- **Orchestrator** (root session) coordinates by spawning scoped workers

Prevents wrong-directory mistakes, accidental cross-repo edits, and context pollution.

### Task System

`_tasks/` stores task files that persist planning context between sessions. Combines two ideas:

- **Prefix routing** — filenames scope work to repos (`fe-1-auth-ui.md`, `x-3-migration.md`)
- **Context distillation** — tasks embed API surfaces, types, schemas from relevant repos

**Flow:**
1. `/create-task Add Google OAuth` → creates `_tasks/x-1-google-oauth.md`
2. Task auto-gathers context from relevant repos
3. Hand task file to a scoped worker agent
4. Worker has full context without re-exploring

**Naming:** `<prefix>-<number>-<slug>.md`
- Repo prefix (from `repos/repos.yaml`) → single repo
- `x` → cross-cutting, multiple repos

See `_tasks/README.md` for the full format.

### Workspace Manifest (repos.yaml)

Declarative source of truth for which repos and local folders belong here. Lives at `repos/repos.yaml`.

```yaml
repos:
  # Git repo (cloned, gitignored)
  - name: frontend          # Directory under repos/
    url: git@...             # Clone URL (required for git)
    branch: main             # Branch to track (git only)
    prefix: fe               # Task routing prefix
    description: React app   # Summary

  # Local source folder (tracked by root repo)
  - name: shared-config     # Directory under repos/
    type: local              # No git, no clone
    prefix: cfg              # Task routing prefix
    description: Shared config and scripts
```

- `type` — `git` (default) or `local`
- `url` — required for git, omitted for local
- `branch` — git only (default: `main`)
- `prefix` — connects to task routing
- `/pull-all-repos` — hydrate from manifest
- `/add-repository` — add one + update manifest

### Cross-Repo PR Linking

`/pr-all-repos` automates multi-repo PRs:
1. Creates PRs in each repo with a non-default branch
2. Cross-links all PRs as siblings in descriptions
3. Match branch names to ticket IDs (e.g., `feature/ENG-123`)

## MCP Servers

Configured in both `.mcp.json` and `.vscode/mcp.json`:

| Server | Purpose |
|--------|---------|
| [Context7](https://github.com/upstash/context7) | Library/framework docs |
| [Playwright](https://github.com/microsoft/playwright-mcp) | Browser automation |

## Cross-Tool Sync (Claude Code + GitHub Copilot)

This workspace strives for cross-compatibility between Claude Code and GitHub Copilot. See [Claude + Copilot Cross-Compatibility](https://raffertyuy.com/raztype/claude-copilot-xcompatibility/) for the approach.

| Feature | Claude Code | GitHub Copilot | VS Code |
|---------|-------------|----------------|---------|
| MCP servers | `.mcp.json` | `.vscode/mcp.json` | `.vscode/mcp.json` |
| Instructions | `CLAUDE.md` | `.github/copilot-instructions.md` | -- |
| Agents | `.claude/agents/` | `.github/agents/` | -- |
| Rules | `.claude/rules/` | `.github/instructions/` | -- |
| Snippets | `.claude/prompt-snippets/` | -- | -- |
| Skills | `.claude/skills/` | -- | -- |
| Settings | `.claude/settings.json` | `.github/copilot-settings.json` | `.vscode/settings.json` |

## Customization

- **Prompt snippets** — shared by 2+ features. One-off? Inline it.
- **Rules** — auto-apply by file glob. Good for language-specific standards.
- **Agents** — specialized personas for specific tasks.
- **MCP servers** — always add to both `.mcp.json` and `.vscode/mcp.json`.
- **Per-repo instructions** — repos can have own `CLAUDE.md`, `AGENTS.md`, `.github/copilot-instructions.md`.

## Prior Art & Inspiration

This template builds on ideas from the community. Specific features we adopted and where they came from:

| Feature | Inspired by |
|---------|-------------|
| Read/write separation (explorer + worker agents) | [ttal](https://dev.to/neil_agentic/how-i-manage-15-repos-with-claude-code-without-losing-my-mind-2ood) — neil_agentic's read/write split across 15+ repos |
| Workspace manifest (`repos.yaml`) | [Virtual Monorepo](https://medium.com/devops-ai/the-virtual-monorepo-pattern-how-i-gave-claude-code-full-system-context-across-35-repos-43b310c97db8) — Owen Zanzal's `.repos` clone script; [Superblocks](https://www.superblocks.com/blog/a-single-dev-workspace-for-ai-agents) `repos.yaml` manifest |
| Cross-repo PR linking (`/pr-all-repos`) | [Superblocks](https://www.superblocks.com/blog/a-single-dev-workspace-for-ai-agents) — `just pr` with sibling PR URL injection |
| Task system with prefix routing | [Spine Pattern](https://tsoporan.com/blog/spine-pattern-multi-repo-ai-development/) — Titus Soporan's prefix-based task scoping |
| Context distillation in task files | [Context from Internal Repos](https://elite-ai-assisted-coding.dev/p/context-from-internal-git-repos) — CI/CD-driven context extraction |

Also worth reading:
- [Polyrepo Synthesis](https://rajiv.com/blog/2025/11/30/polyrepo-synthesis-synthesis-coding-across-multiple-repositories-with-claude-code-in-visual-studio-code/) — Rajiv Pant, unified VS Code workspace across repos
- [The Code Agent Orchestra](https://addyosmani.com/blog/code-agent-orchestra/) — Addy Osmani, multi-agent coordination patterns

## License

MIT
