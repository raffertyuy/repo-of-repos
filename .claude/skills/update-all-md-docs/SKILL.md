---
name: update-all-md-docs
description: Review and update all markdown docs in the repo-of-repos (excluding cloned repos)
user-invocable: true
---

Review and update all `.md` files in the repo-of-repos workspace to ensure they are accurate and consistent with the current state of the project.

## Scope

- **Include**: All `*.md` files in the root and its subdirectories (`.claude/`, `.github/`, `.vscode/`, `repos/*.md`)
- **Read but don't edit**: Files inside cloned repos (`repos/*/`) — read these to inform updates to `repos/*.md` files, but never modify them

## Steps

1. **Discover** all in-scope `.md` files using glob patterns. Also list directories in `repos/*/` to identify cloned repos.

2. **Read each file** and check for:
   - Outdated references to files, directories, or features that no longer exist
   - Missing references to new files, directories, or features that have been added
   - Inconsistencies between documents (e.g., README.md describes a structure that doesn't match the actual file tree)
   - Stale MCP server lists, skill lists, agent lists, or config tables
   - Broken relative links

3. **Update each file** to reflect the current state:
   - Sync the file tree in README.md with the actual directory structure
   - Ensure MCP server tables match what's in `.mcp.json` and `.vscode/mcp.json`
   - Ensure skill/agent/rule references are complete and accurate
   - Update `repos/repos.md` by reading into each cloned repo (`repos/*/`) to get current READMEs, tech stacks, directory structures, and agent instruction files — then regenerate the entries using the `/analyze-repositories` format
   - Keep the existing tone and structure of each file — don't rewrite from scratch

4. **Report** a summary of what was changed and why.
