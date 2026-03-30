# repo-of-repos

A starter template for working with **multiple git repos as a single AI-powered workspace**.

## The Problem

Most real projects span multiple repositories -- frontend, backend, microservices, infrastructure, etc. Agentic AI tools (Claude Code, GitHub Copilot, VS Code) work best when they can see everything in one folder, but you don't want to merge all your repos into a monorepo.

## The Solution

Clone your related repos into `repos/` and let the root-level agentic configs provide cross-repo context. Each repo stays independent (its own git history, CI, deploys), but your AI tools get the full picture.

```
.
├── CLAUDE.md                          # Project-wide instructions for Claude Code
├── .mcp.json                          # MCP servers (Claude Code)
├── .vscode/
│   └── mcp.json                       # MCP servers (VS Code / Copilot)
├── .claude/
│   ├── settings.json                  # Claude Code settings (shared)
│   ├── agents/
│   │   └── reviewer.md                # Code review agent
│   ├── rules/
│   │   └── frontend.md                # Auto-applied rules for frontend files
│   ├── skills/
│   │   ├── commit/                    # /commit slash command
│   │   └── analyze-repositories/      # /analyze-repositories slash command
│   └── prompt-snippets/               # Reusable instructions shared across features
│       ├── coding-standards.md
│       ├── commit-message.md
│       └── review-standards.md
├── .github/
│   ├── agents/
│   │   └── reviewer.md                # Copilot code review agent
│   └── instructions/
│       └── frontend.instructions.md   # Copilot scoped instructions for frontend
├── repos/
│   ├── repos.md                       # Auto-generated descriptions of each repo
│   ├── frontend/                      # (cloned repo, gitignored)
│   ├── backend-api/                   # (cloned repo, gitignored)
│   └── infra/                         # (cloned repo, gitignored)
└── .gitignore
```

## Getting Started

1. **Use this template** -- click "Use this template" on GitHub or clone the repo
2. **Clone your repos** into `repos/`:
   ```sh
   git clone git@github.com:your-org/frontend.git repos/frontend
   git clone git@github.com:your-org/backend-api.git repos/backend-api
   git clone git@github.com:your-org/infra.git repos/infra
   ```
3. **Run `/analyze-repositories`** in Claude Code to auto-generate `repos/repos.md` with descriptions of each repo
4. **Customize** the agentic configs to match your project's standards

The cloned repos are gitignored (`repos/*/`) so this wrapper repo stays lightweight. Files directly in `repos/` (like `repos.md`) are tracked.

## MCP Servers

Pre-configured in both `.mcp.json` and `.vscode/mcp.json`:

| Server | Purpose |
| --- | --- |
| [Context7](https://github.com/upstash/context7) | Up-to-date library/framework documentation |
| [Playwright](https://github.com/microsoft/playwright-mcp) | Browser automation and testing |

## How Features Map Across Tools

| Feature | Claude Code | GitHub Copilot | VS Code |
| --- | --- | --- | --- |
| MCP servers | `.mcp.json` | `.vscode/mcp.json` | `.vscode/mcp.json` |
| Project instructions | `CLAUDE.md` | `.github/copilot-instructions.md` | -- |
| Agents | `.claude/agents/` | `.github/agents/` | -- |
| Scoped instructions | `.claude/rules/` | `.github/instructions/` | -- |
| Reusable snippets | `.claude/prompt-snippets/` | -- | -- |
| Skills / commands | `.claude/skills/` | -- | -- |
| Settings | `.claude/settings.json` | `.github/copilot-settings.json` | `.vscode/settings.json` |

## Customization Tips

- **Prompt snippets** (`.claude/prompt-snippets/`) are for instructions shared by 2+ agentic features. If an instruction is only used once, inline it directly in the consuming file.
- **Rules** (`.claude/rules/`) auto-apply based on file path globs -- useful for language/framework-specific standards.
- **Agents** (`.claude/agents/`, `.github/agents/`) are specialized personas you can invoke for specific tasks like code review.
- **MCP servers** should always be added to both `.mcp.json` and `.vscode/mcp.json` to keep tooling in sync.

## License

MIT
