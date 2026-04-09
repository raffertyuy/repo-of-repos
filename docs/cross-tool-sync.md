# Cross-Tool Sync (Claude Code + GitHub Copilot)

This workspace strives for cross-compatibility between Claude Code and GitHub Copilot. Both tools see the same repos and follow the same standards.

For the full approach, see [Claude + Copilot Cross-Compatibility](https://raffertyuy.com/raztype/claude-copilot-xcompatibility/).

## Where Things Live

| Feature | Claude Code | GitHub Copilot | VS Code |
|---------|-------------|----------------|---------|
| MCP servers | `.mcp.json` | `.vscode/mcp.json` | `.vscode/mcp.json` |
| Instructions | `CLAUDE.md` | `.github/copilot-instructions.md` | — |
| Agents | `.claude/agents/` | `.github/agents/` | — |
| Rules | `.claude/rules/` | `.github/instructions/` | — |
| Snippets | `.claude/prompt-snippets/` | — | — |
| Skills | `.claude/skills/` | — | — |
| Settings | `.claude/settings.json` | `.github/copilot-settings.json` | `.vscode/settings.json` |

## Sync Rules

1. **MCP servers** — any server added to `.mcp.json` must also go in `.vscode/mcp.json` (and vice versa). The schema differs slightly:
   - `.mcp.json` uses `"mcpServers": { ... }`
   - `.vscode/mcp.json` uses `"servers": { ... }`
   - The server definition inside (command, args, env) is identical.

2. **Instructions/prompts** — when creating or updating agent instructions, consider whether an equivalent should exist for the other tool. Shared standards (coding style, review guidelines) should be consistent.

3. **Always check both files** before making changes — don't assume they're in sync.

4. **Test after syncing** — verify the MCP server or config works in the target tool.

## Per-Repo Instructions

Each repo in `repos/` can have its own instructions:

| File | Tool | Purpose |
|------|------|---------|
| `CLAUDE.md` | Claude Code | Repo-specific instructions |
| `AGENTS.md` | Claude Code / Copilot | Shared agent instructions |
| `.github/copilot-instructions.md` | GitHub Copilot | Copilot-specific instructions |

These supplement (not override) the root-level instructions.
