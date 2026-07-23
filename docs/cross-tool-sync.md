# Cross-Tool Sync (Claude Code + GitHub Copilot + OpenAI Codex)

This workspace is cross-compatible across **Claude Code**, **GitHub Copilot**, and **OpenAI Codex**. All three see the same repos, the same instructions, and the same standards — without duplicating content.

For the full approach, see [Claude + Copilot + Codex Cross-Compatibility](https://raffertyuy.com/raztype/claude-copilot-codex-cross-compatibility/).

## The Principle

Every feature has **one canonical file**. Other tools read it natively, or through a thin pointer/stub. Never maintain the same content in two places.

## Where Things Live

| Feature | Canonical | Claude Code | GitHub Copilot | OpenAI Codex |
| ------- | --------- | ----------- | -------------- | ------------ |
| Instructions | `AGENTS.md` | Via `CLAUDE.md` → `@AGENTS.md` | Native | Native |
| Skills | `.agents/skills/<name>/SKILL.md` | Via stubs in `.claude/skills/` | Native | Native |
| Custom agents | `.claude/agents/*.md` | Native | Native | ❌ Gap (needs `.codex/agents/` TOML) |
| Path-scoped rules | `.claude/rules/*.md` | Native | Native (VS Code) | ❌ Gap (needs nested `AGENTS.md`) |
| MCP servers | `.mcp.json` | Native | Native (VS Code 1.118+) | Mirror in `.codex/config.toml` |
| Settings | `.claude/settings.json` | Native | — | `.codex/config.toml` |

## How Each Pointer Works

### Instructions: `CLAUDE.md` → `AGENTS.md`

`AGENTS.md` holds ALL workspace instructions. `CLAUDE.md` contains exactly one line:

```text
@AGENTS.md
```

Claude Code expands the `@` import; Copilot and Codex read `AGENTS.md` directly. Never write instructions into `CLAUDE.md`, and don't create `.github/copilot-instructions.md` — duplicate instruction files waste context.

### Skills: canonical + stub

Canonical skill content lives in `.agents/skills/<name>/SKILL.md` (read natively by Copilot and Codex). Each skill has a stub at `.claude/skills/<name>/SKILL.md`:

```markdown
---
name: my-skill
description: What the skill does
user-invocable: true
origin: template
---

@../../../.agents/skills/my-skill/SKILL.md
```

The frontmatter is duplicated because Claude Code reads only the stub's frontmatter for skill discovery. The `@` reference pulls in the canonical body at invocation time.

**When editing a skill**: body changes go to the canonical file only. Frontmatter changes (name, description) must be applied to BOTH files. New skills need both files created.

### MCP servers: `.mcp.json` + `.codex/config.toml`

`.mcp.json` is read by Claude Code, Copilot CLI, and VS Code 1.118+. Codex needs the same servers mirrored in `.codex/config.toml`:

```json
// .mcp.json
{
  "mcpServers": {
    "playwright": { "command": "npx", "args": ["-y", "@playwright/mcp@latest"] }
  }
}
```

```toml
# .codex/config.toml
[mcp_servers.playwright]
command = "npx"
args = ["-y", "@playwright/mcp@latest"]
```

Same command, args, and env — different wrapper. Any server added to one file must be added to the other.

> **Older VS Code** (< 1.118) doesn't read `.mcp.json`. If MCP servers don't show up in VS Code, create `.vscode/mcp.json` with a `"servers"` key mirroring `.mcp.json`.

### Custom agents: `.claude/agents/`

VS Code and Copilot CLI read Claude-format agents from `.claude/agents/` directly — one file serves both. Codex doesn't read this format (it uses `.codex/agents/` TOML files); this is a known gap. If you need an agent in Codex, create a TOML wrapper that references the shared prompt content.

### Path-scoped rules: `.claude/rules/`

`.claude/rules/*.md` with `paths:` frontmatter is read by Claude Code and VS Code Copilot. Codex doesn't support this format — its equivalent is a nested `AGENTS.md` in the target folder. Add one only if you actively use Codex on those paths.

## Sync Rules

1. **One canonical location per feature** — check the table above before creating any agentic config file.
2. **MCP servers** — always update `.mcp.json` AND `.codex/config.toml` together.
3. **Skills** — body edits touch only `.agents/skills/`; frontmatter edits touch both canonical and stub.
4. **Always check both files** before making changes — don't assume they're in sync.
5. **Test after syncing** — verify the config works in the target tool.

## Per-Repo Instructions

Each repo in `repos/` can have its own instructions:

| File | Tool | Purpose |
| ---- | ---- | ------- |
| `AGENTS.md` | All tools (canonical) | Repo-specific agent instructions |
| `CLAUDE.md` | Claude Code | Claude-specific instructions (ideally an `@AGENTS.md` pointer) |
| `.github/copilot-instructions.md` | GitHub Copilot | Legacy Copilot instructions |

These supplement (not override) the root-level instructions.
