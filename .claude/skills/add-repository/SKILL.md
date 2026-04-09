---
name: add-repository
description: Clone a git repo or register a local folder into repos/ and update repos.md with its description
user-invocable: true
---

Clone a git repository or register a local source folder into `repos/` and add its entry to `repos/repos.yaml` and `repos/repos.md`.

The user will provide a git URL, local path, or directory name. For example:
- `/add-repository git@github.com:org/my-app.git` — clone a git repo
- `/add-repository https://github.com/org/my-app.git` — clone a git repo
- `/add-repository /path/to/local/repo` — clone a local git repo
- `/add-repository --local shared-config` — create/register a local source folder
- `/add-repository --local repos/my-scripts` — register an existing local folder

## Detecting Type

- If the user passes `--local` flag, treat as a local source folder
- If the argument is a git URL (contains `git@`, `https://`, `.git`, or points to a directory with `.git/`), treat as a git repo
- If ambiguous, ask the user

## Git Repo Flow

1. **Clone** the repo into `repos/` using `git clone <url> repos/<repo-name>`. Infer the repo name from the URL (e.g., `my-app` from `git@github.com:org/my-app.git`). If a directory with that name already exists in `repos/`, inform the user and stop.

2. **Analyze** the newly cloned repo (see Analysis section below).

3. **Update `repos/repos.yaml`** by adding an entry:

   ```yaml
   - name: <repo-name>
     url: <git remote URL>
     branch: <current branch>
     prefix: <short prefix>
     description: <1-line summary>
   ```

4. **Update `.gitignore`** — add `repos/<repo-name>/` to `.gitignore` if not already present.

5. **Update `repos/repos.md`** (see repos.md section below).

## Local Folder Flow

1. **Create or verify** the directory:
   - If `repos/<name>/` doesn't exist, create it with `mkdir -p repos/<name>`
   - If it already exists, verify it does NOT have a `.git/` directory (otherwise suggest adding as a git repo)
   - If a matching entry already exists in `repos/repos.yaml`, inform the user and stop

2. **Analyze** the folder if it has contents (see Analysis section below).

3. **Update `repos/repos.yaml`** by adding an entry:

   ```yaml
   - name: <folder-name>
     type: local
     prefix: <short prefix>
     description: <1-line summary>
   ```

4. **Ensure NOT in `.gitignore`** — if `repos/<folder-name>/` appears in `.gitignore`, remove it. Local folders are tracked by the root repo.

5. **Update `repos/repos.md`** (see repos.md section below).

## Analysis

For both git repos and local folders:

- Read its README.md (or README) if one exists
- Check its `package.json`, `Cargo.toml`, `go.mod`, `pyproject.toml`, or equivalent to understand the tech stack
- For git repos: read its git remote URL (`git -C repos/<name> remote get-url origin`)
- Briefly look at the top-level directory structure
- Check which agent instruction files exist: `CLAUDE.md`, `AGENTS.md`, `.github/copilot-instructions.md`

## repos.md Update

Append a new section to `repos/repos.md`:

```markdown
## <name>

- **Type**: git | local
- **Remote**: <git remote URL> *(git repos only)*
- **Tech stack**: <languages, frameworks, key dependencies>
- **Description**: <1-2 sentence summary from README or inferred from code>
- **Key paths**: <notable top-level directories and what they contain>
- **Agent instructions**: <list which of CLAUDE.md, AGENTS.md, .github/copilot-instructions.md exist, or "None">
```

Omit the **Remote** line for local entries. If `repos/repos.md` still has the placeholder comment (`<!-- No repositories found... -->`), remove it before appending.

If `repos/repos.yaml` has an empty list (`repos: []`), replace it with the new entry. Otherwise append to the list. Infer a sensible `prefix` from the name (first 2-4 characters or a natural abbreviation). If unsure, ask the user.
