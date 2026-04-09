# Workspace Manifest (repos.yaml)

`repos/repos.yaml` is the declarative source of truth for which repos and local folders belong in this workspace.

## Format

```yaml
repos:
  # Git repo (cloned, gitignored, has its own git history)
  - name: frontend          # Directory under repos/ (required)
    url: git@...             # Clone URL (required for git)
    branch: main             # Branch to track (default: main)
    prefix: fe               # Short prefix (optional)
    description: React app   # One-line summary (optional)

  # Local source folder (tracked by root repo)
  - name: shared-config
    type: local              # No git, no clone
    prefix: cfg
    description: Shared config and scripts

  # Local source folder (gitignored)
  - name: scratch
    type: local
    gitignore: true          # Not committed to root repo
    prefix: scr
    description: Scratch space
```

## Field Reference

| Field | Required | Default | Applies to | Description |
|-------|----------|---------|------------|-------------|
| `name` | Yes | — | Both | Directory name under `repos/` |
| `type` | No | `git` | Both | `"git"` or `"local"` |
| `url` | Git only | — | Git | Clone URL |
| `branch` | No | `main` | Git | Branch to track |
| `gitignore` | No | `false` | Local | `true` to gitignore the folder |
| `prefix` | No | — | Both | Short prefix for the repo (used in references) |
| `description` | No | — | Both | One-line summary (auto-filled by `/pull-all-repos`) |

## Commands

- `/pull-all-repos` — clone/pull all entries from the manifest
- `/add-repository <url>` — add a git repo and update the manifest
- `/add-repository --local <name>` — add a local folder and update the manifest
- `/remove-repository <name>` — remove an entry from the manifest
- `/clean-repos` — remove stale entries whose directories no longer exist

## See Also

- [Adding Repos](adding-repos.md) — step-by-step guide for adding repos and local folders
