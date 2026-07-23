---
name: pull-all-repos
description: Clone/pull git repos, verify local folders, auto-register orphans, and regenerate repos.md
user-invocable: true
origin: template
---

# Sync Workspace

Read `repos/repos.yaml` at the project root and ensure all declared repositories and local source folders are present and up to date.

## Order of Operations

1. **Parse** `repos/repos.yaml` — read the `repos:` list
2. **Clone missing git repos** — for each `type: git` (or unset type) entry where `repos/<name>/` doesn't exist, clone it
3. **Pull existing git repos** — for each git entry where `repos/<name>/` already exists, fetch and pull
4. **Verify local entries** — for each `type: local` entry, verify `repos/<name>/` exists on disk
5. **Detect and register orphans** — find directories in `repos/` not in the manifest, auto-detect type, analyze, and add to `repos/repos.yaml`
6. **Sync .gitignore** — ensure each git repo has an ignore entry; local entries must NOT be in `.gitignore`
7. **Regenerate repos.md** — update `repos/repos.md` with current repo/folder analysis

## Parsing repos/repos.yaml

Read `repos/repos.yaml` from the project root. Each entry has:

- `name` (required) — directory name under `repos/`
- `type` (optional, default: `git`) — `"git"` or `"local"`
- `url` (required for git, ignored for local) — git clone URL
- `branch` (optional, default: `main`, git only) — branch to track
- `prefix` (optional) — short prefix for the repo (used in references)
- `description` (optional) — one-line summary

If `repos/repos.yaml` has an empty list (`repos: []`), skip steps 2–4 and proceed directly to step 5 (orphan detection). The manifest being empty does NOT mean the workspace is empty — there may be repos on disk that need to be registered.

## Cloning Missing Git Repos

For each entry where `type` is `git` (or unset) and `repos/<name>/` does NOT exist:

1. Run `git clone <url> repos/<name>`
2. If a `branch` is specified and it's not the default, run `git -C repos/<name> checkout <branch>`
3. Report: cloned successfully or error

## Pulling Existing Git Repos

For each entry where `type` is `git` (or unset) and `repos/<name>/` already exists:

1. Verify it's a git repo (has `.git` directory)
2. Run `git -C repos/<name> fetch --all --prune`
3. Check the current branch: `git -C repos/<name> branch --show-current`
4. If the branch has an upstream, run `git -C repos/<name> pull --ff-only`
   - If fast-forward fails, report and skip — **never force-pull**
   - If uncommitted changes block the pull, report and skip
5. Report: updated (with new commit count), already up to date, or error

## Verifying Local Entries

For each entry where `type` is `local`:

1. Check if `repos/<name>/` exists on disk
2. If it exists, report: present
3. If it does NOT exist, create the directory (`mkdir -p repos/<name>`) and report: created empty directory — user should add source files

Local entries are never cloned, pulled, or synced from a remote. They are source folders managed directly.

## Detecting and Registering Orphans

List directories in `repos/` and compare against `repos/repos.yaml` entries. For any directory that exists on disk but isn't in the manifest:

1. **Auto-detect type** — check if the directory has a `.git/` subdirectory
   - Has `.git/` → `type: git`
   - No `.git/` → `type: local`

2. **For git orphans**:
   - Read the remote URL: `git -C repos/<name> remote get-url origin`
   - Read the current branch: `git -C repos/<name> branch --show-current`
   - Add to `repos/repos.yaml`:
     ```yaml
     - name: <name>
       url: <remote URL>
       branch: <current branch>
       prefix: <inferred from name>
       description: <1-line summary from analysis>
     ```

3. **For local orphans**:
   - Ask the user: **"Should `repos/<name>/` be tracked by git, or gitignored?"**
   - Add to `repos/repos.yaml`:
     ```yaml
     - name: <name>
       type: local
       gitignore: <true|false based on user's answer>
       prefix: <inferred from name>
       description: <1-line summary from analysis>
     ```

4. **Analyze** each orphan the same way as existing entries — read README, detect tech stack, scan directory structure (see Regenerate repos.md section)

5. **Infer a prefix** from the directory name (first 2-4 characters or a natural abbreviation, e.g., `frontend` → `fe`, `shared-config` → `cfg`). Avoid collisions with existing prefixes. If unsure, use the full name truncated.

6. **Report** what was registered:
   > ✅ Registered `repos/<name>` as `type: git` (remote: `<url>`) — added to `repos/repos.yaml`
   > ✅ Registered `repos/<name>` as `type: local` — added to `repos/repos.yaml`

Do NOT delete any directories. Registration is additive only.

## Syncing .gitignore

After processing all entries, ensure the root `.gitignore` is correct:

1. Read the current `.gitignore`
2. For each **git** entry, ensure `repos/<name>/` is listed in `.gitignore` (add if missing) — git repos are always gitignored
3. For each **local** entry, check the `gitignore` field in `repos/repos.yaml`:
   - If `gitignore: true` → ensure `repos/<name>/` IS in `.gitignore`
   - If `gitignore: false` (or unset) → ensure `repos/<name>/` is NOT in `.gitignore`
   - If the entry has no `gitignore` field and the directory was just registered as an orphan, the user was already asked in the orphan detection step — use their answer
4. Keep the header comments intact. Add new entries below the header block.

## Regenerate repos.md

After all sync operations, regenerate `repos/repos.md`. For each entry (git or local):

1. Read its README.md (or README) if one exists
2. Check `package.json`, `Cargo.toml`, `go.mod`, `pyproject.toml`, or equivalent for tech stack
3. For git entries: read git remote URL with `git -C repos/<name> remote get-url origin`
4. Briefly scan top-level directory structure
5. Check which agent instruction files exist: `CLAUDE.md`, `AGENTS.md`, `.github/copilot-instructions.md`

Write `repos/repos.md` with this format:

```markdown
# Repositories

This file describes each repository and local source folder in `repos/`. Auto-generated by `/pull-all-repos`. Do not edit manually.

## <repo-name>

- **Type**: git | local
- **Remote**: <git remote URL> *(git only)*
- **Tech stack**: <languages, frameworks, key dependencies>
- **Description**: <1-2 sentence summary>
- **Key paths**: <notable top-level directories>
- **Agent instructions**: <which of CLAUDE.md, AGENTS.md, .github/copilot-instructions.md exist, or "None">
```

Omit the **Remote** line for local entries. If `repos/` has no directories, keep the placeholder comment.

## Summary

Report a table at the end:

| Entry | Type | Status |
|-------|------|--------|
| `<name>` | git | Cloned / Updated (N new commits) / Already up to date / Error: `<reason>` |
| `<name>` | local | Present / Created empty directory |
| `<name>` | git | ✅ Registered (was orphan) |
| `<name>` | local | ✅ Registered (was orphan) |

## Notes

- Never force-pull or reset sub-repos
- Never discard uncommitted local changes
- Run clones and pulls sequentially to avoid credential prompt conflicts
- If `repos/repos.yaml` doesn't exist, create it.
