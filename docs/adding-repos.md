# Adding Repos and Local Folders

The `repos/` directory holds everything your project is made of — git repos and local source folders. This guide covers all the ways to add them.

## Git Repos vs Local Folders

| | Git repos | Local folders |
|---|---|---|
| **Has own `.git`** | Yes | No |
| **Tracked by root repo** | No (gitignored) | Yes (default) |
| **Cloned/pulled by `/pull-all-repos`** | Yes | Verified/created |
| **Committed by `/commit-all-repos`** | Per sub-repo | Via root `/commit` |
| **PRs via `/pr-all-repos`** | Yes | Skipped |

## Adding Git Repos

### Option A: Manifest (recommended)

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

### Option B: One at a time

```
/add-repository <url>
```

This clones the repo, analyzes it, and updates `repos/repos.yaml` and `repos/repos.md` automatically.

## Adding Local Source Folders

For source code that doesn't belong to any git repo — scripts, shared config, scratch projects, prototypes.

### Option A: Slash command

```
/add-repository --local shared-config
```

This will:
1. Create `repos/shared-config/` if it doesn't exist
2. Add a `type: local` entry to `repos/repos.yaml`
3. Update `repos/repos.md` with a description
4. Ask whether the folder should be **tracked by git** (default) or **gitignored**

Then add your source files into `repos/shared-config/`.

### Option B: Manifest

Add a `type: local` entry to `repos/repos.yaml`:

```yaml
repos:
  - name: shared-config
    type: local
    prefix: cfg
    description: Shared configuration and scripts
```

Run `/pull-all-repos` — it will create the directory if missing and register it.

### Option C: Manual

1. Create the folder: `mkdir repos/my-project`
2. Add your source files
3. Add an entry to `repos/repos.yaml` with `type: local`
4. Run `/pull-all-repos` to regenerate `repos/repos.md`

## How Git Tracking Works

- **Git repos** are always gitignored. Each keeps its own git history, CI, and deploys.
- **Local folders** default to being tracked by the root repo. You can opt out by setting `gitignore: true` in `repos.yaml`.

## See Also

- [Workspace Manifest](workspace-manifest.md) — full `repos.yaml` field reference
- `repos/repos.md` — auto-generated descriptions of all entries
