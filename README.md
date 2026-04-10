# repo-of-repos

> **Based on [raffertyuy/repo-of-repos](https://github.com/raffertyuy/repo-of-repos).** This is an implementation of that upstream template. Periodically run `/sync-template` to pick up improvements.

**Meet Tony Stark** — the AI agent that runs this workspace. Call him **Tony**, **Mr. Stark**, or just **T**. Why Tony Stark? Because like the real deal, he works on multiple projects at the same time — wiring services, orchestrating repos, and delegating to subagents without breaking a sweat. One workspace, many repos, one genius coordinating it all.

A starter template for working with **multiple git repos and local source folders as a single AI-powered workspace**.

## The Problem

Real projects span many repos. AI tools work best when they see everything in one place. You don't want a monorepo.

## The Solution

Clone repos into `repos/`. Put local source code there too. Root-level agentic configs provide cross-repo context. Git repos stay independent (own git history, CI, deploys). Local folders are tracked by the root repo. Your AI gets the full picture.

This pattern goes by many names — "Virtual Monorepo," "Spine Pattern," "Polyrepo Synthesis." This template packages the best ideas into a ready-to-use workspace.

## Getting Started

### 1. Use This Template

Click "Use this template" on GitHub, or clone directly.

### 2. Add Your Repos

Edit `repos/repos.yaml` to declare your repos:

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

Run `/pull-all-repos` to clone everything. Or add one at a time with `/add-repository <url>`.

You can also add local source folders (scripts, shared config, prototypes) that don't have their own git repo. See [docs/adding-repos.md](docs/adding-repos.md) for all options.

### 3. Customize

- `CLAUDE.md` — add project context, architecture notes, data flows
- `.claude/rules/` — add rules for your languages/frameworks
- `.claude/prompt-snippets/` — tune shared instructions

### 4. Start Working

You're ready. Use the plan-implement workflow below, or just start talking to the AI.

## Plan-Implement Workflow

This is the core workflow for getting things built. Instead of jumping straight into code, you plan first, then execute. The plan tracks progress and documents what actually happened.

Inspired by the [Plan-Implement-Run pattern](https://raffertyuy.com/raztype/vibe-coding-plan-implement-run/).

### Step 1: Plan

```
/plan Add Google OAuth to the login flow
```

This creates a plan file in `_plans/` with:

- **Context** gathered from your repos (API surfaces, types, key files)
- **Steps** as markdown checkboxes (`- [ ]`) with pseudocode
- **File references** so the implementing agent knows where to work

Example output: `_plans/20260409-google-oauth.plan.md`

```markdown
---
status: draft
repos: [frontend, backend-api]
created: 2026-04-09
---

# Plan: Add Google OAuth

## Steps

- [ ] Step 1: Add Google OAuth endpoint
  - **Task**: Create POST /api/auth/google with token validation
  - **Files**: `repos/backend-api/src/routes/auth.ts`
  - **Pseudocode**: Validate Google token, create session, return AuthResponse

- [ ] Step 2: Add Google sign-in button
  - **Task**: Wire OAuth redirect flow in the login form
  - **Files**: `repos/frontend/src/components/LoginForm.tsx`
  - **Pseudocode**: Render Google button, handle callback, store token

- [ ] Step 3: Validate
  - **Task**: Run tests, verify the full OAuth flow works
```

Review the plan. Adjust steps if needed. This is your chance to catch issues before any code is written.

### Step 2: Implement

```
/implement google-oauth
```

This executes the plan:

1. Reads the plan and sets status to `in-progress`
2. Implements each step, writing the actual code
3. Checks off `- [ ]` to `- [x]` after each step
4. Adds notes when something unexpected comes up
5. Runs the app and tests
6. Documents any fixes as new steps
7. Sets status to `completed` when everything passes

After implementation, the plan looks like this:

```markdown
- [x] Step 1: Add Google OAuth endpoint
  - **Task**: Create POST /api/auth/google with token validation
  - **Files**: `repos/backend-api/src/routes/auth.ts`
  - **Notes**: Used existing session middleware instead of creating new one.

- [x] Step 2: Add Google sign-in button
  - **Task**: Wire OAuth redirect flow in the login form
  - **Files**: `repos/frontend/src/components/LoginForm.tsx`

- [x] Step 3: Validate
  - **Task**: Run tests, verify the full OAuth flow works

- [x] Fix: Token parser updated for OAuth v2 response format
  - **Notes**: Google changed token format in v2. Updated parser to handle both.
```

The plan becomes a record of what was planned AND what actually happened — useful for code reviews, handoffs, and future reference.

### Why Plan First?

- **Catches mistakes early** — cheaper to fix a plan than to rewrite code
- **Session continuity** — if you close the chat, a new session picks up where you left off
- **Visible progress** — checkboxes show exactly what's done and what's left
- **Self-documenting** — decisions and fixes are recorded as they happen

See `_plans/README.md` for the full plan file format.

## Project Structure

```
.
├── CLAUDE.md                          # Root instructions for Claude Code
├── TEMPLATE_VERSION                   # Current template version
├── TEMPLATE_CHANGELOG.md              # Template changelog
├── .mcp.json                          # MCP servers (Claude Code)
├── .vscode/
│   └── mcp.json                       # MCP servers (VS Code / Copilot)
├── .claude/
│   ├── settings.json                  # Claude Code settings
│   ├── agents/                        # explorer, worker, reviewer
│   ├── rules/                         # Auto-applied rules by file type
│   ├── skills/                        # Slash commands (see below)
│   └── prompt-snippets/               # Shared instructions
├── .github/
│   ├── agents/                        # Copilot equivalents
│   └── instructions/                  # Copilot rules
├── _plans/                            # Implementation plans
│   └── README.md
├── docs/                              # Detailed reference docs
├── repos/
│   ├── repos.yaml                     # Workspace manifest
│   ├── repos.md                       # Auto-generated descriptions
│   └── <your-repos>/                  # Your code lives here
└── .gitignore
```

## Slash Commands

Invoked in Claude Code with `/<name>`. Defined in `.claude/skills/`.

### Workspace

| Command | What it does |
|---------|-------------|
| `/pull-all-repos` | Clone/pull all repos, verify local folders, regenerate `repos.md` |
| `/add-repository <url>` | Clone a git repo and register it |
| `/add-repository --local <name>` | Create/register a local source folder |
| `/remove-repository <name>` | Remove a repo/folder from config |
| `/clean-repos` | Remove stale entries whose directories no longer exist |

### Plan & Implement

| Command | What it does |
|---------|-------------|
| `/plan <description>` | Create an implementation plan with steps, context, and pseudocode |
| `/implement [plan-file]` | Execute a plan — build, test, fix, and update the plan as you go |

### Git

| Command | What it does |
|---------|-------------|
| `/commit` | Stage, commit, and push root workspace only (excludes `repos/`) |
| `/commit-all-repos` | Stage, commit, and push each sub-repo, then root workspace |
| `/pr-all-repos` | Create PRs in all sub-repos, cross-link as siblings |

### Docs & Maintenance

| Command | What it does |
|---------|-------------|
| `/update-all-md-docs` | Sync all markdown files with current state |
| `/sync-template` | Pull latest template updates from upstream |

## Agents

The workspace uses three specialized agents. Defined in `.claude/agents/` and mirrored in `.github/agents/` for Copilot.

| Agent | Scope | Purpose |
|-------|-------|---------|
| **explorer** | Read-only, all repos | Trace dependencies, find types, answer "what breaks if I change X?" |
| **worker** | Write-scoped, one repo | Implement changes. Follows the target repo's own instructions. |
| **reviewer** | Read-only | Code review against shared standards. |

**Key rule: reads are cross-repo, writes are single-repo.** The explorer reads across all repos. The worker only writes to one repo at a time. The orchestrator (Tony) coordinates by spawning scoped workers. This prevents wrong-directory mistakes and accidental cross-repo edits.

## MCP Servers

Configured in both `.mcp.json` and `.vscode/mcp.json`:

| Server | Purpose |
|--------|---------|
| [Context7](https://github.com/upstash/context7) | Library/framework docs |
| [Playwright](https://github.com/microsoft/playwright-mcp) | Browser automation |

## Keeping Up to Date

This template evolves. To pull the latest changes:

```
/sync-template
```

This checks your `TEMPLATE_VERSION`, copies updated framework files, and smart-merges files that mix template and project content. It never touches your repos, plans, settings, or MCP config.

See `TEMPLATE_CHANGELOG.md` for what's new in each version.

## Further Reading

| Topic | Where |
|-------|-------|
| Adding repos and local folders | [docs/adding-repos.md](docs/adding-repos.md) |
| Workspace manifest format | [docs/workspace-manifest.md](docs/workspace-manifest.md) |
| Cross-tool sync (Claude Code + Copilot) | [docs/cross-tool-sync.md](docs/cross-tool-sync.md) |
| Plan file format | [_plans/README.md](_plans/README.md) |

## Customization

- **`CLAUDE.md`** — project context, architecture, data flows
- **`.claude/rules/`** — auto-apply rules by file glob (e.g., frontend standards)
- **`.claude/prompt-snippets/`** — shared instructions used by 2+ features
- **Agents** — add or customize in `.claude/agents/`
- **MCP servers** — add to both `.mcp.json` and `.vscode/mcp.json`
- **Per-repo instructions** — repos can have their own `CLAUDE.md`, `AGENTS.md`, or `.github/copilot-instructions.md`

## Prior Art & Inspiration

This template builds on ideas from the community:

| Feature | Inspired by |
|---------|-------------|
| Plan-implement workflow | [Plan-Implement-Run](https://raffertyuy.com/raztype/vibe-coding-plan-implement-run/) — structured plan/implement/run phases with living plan documents |
| Read/write separation (explorer + worker agents) | [ttal](https://dev.to/neil_agentic/how-i-manage-15-repos-with-claude-code-without-losing-my-mind-2ood) — neil_agentic's read/write split across 15+ repos |
| Workspace manifest (`repos.yaml`) | [Virtual Monorepo](https://medium.com/devops-ai/the-virtual-monorepo-pattern-how-i-gave-claude-code-full-system-context-across-35-repos-43b310c97db8) — Owen Zanzal's `.repos` clone script; [Superblocks](https://www.superblocks.com/blog/a-single-dev-workspace-for-ai-agents) `repos.yaml` manifest |
| Cross-repo PR linking (`/pr-all-repos`) | [Superblocks](https://www.superblocks.com/blog/a-single-dev-workspace-for-ai-agents) — `just pr` with sibling PR URL injection |
| Repo-scoped planning | [Spine Pattern](https://tsoporan.com/blog/spine-pattern-multi-repo-ai-development/) — Titus Soporan's prefix-based task scoping |
| Context distillation in plans | [Context from Internal Repos](https://elite-ai-assisted-coding.dev/p/context-from-internal-git-repos) — CI/CD-driven context extraction |
| Cross-tool sync (Claude + Copilot) | [Claude + Copilot Cross-Compatibility](https://raffertyuy.com/raztype/claude-copilot-xcompatibility/) — keeping both tools in sync |

Also worth reading:
- [Polyrepo Synthesis](https://rajiv.com/blog/2025/11/30/polyrepo-synthesis-synthesis-coding-across-multiple-repositories-with-claude-code-in-visual-studio-code/) — Rajiv Pant, unified VS Code workspace across repos
- [The Code Agent Orchestra](https://addyosmani.com/blog/code-agent-orchestra/) — Addy Osmani, multi-agent coordination patterns

## License

MIT
