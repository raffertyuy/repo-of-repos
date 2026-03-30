---
name: add-repository
description: Clone a git repo into repos/ and update repos.md with its description
user-invocable: true
---

Clone a git repository into `repos/` and add its entry to `repos/repos.md`.

The user will provide a git URL or local path. For example:
- `/add-repository git@github.com:org/my-app.git`
- `/add-repository https://github.com/org/my-app.git`
- `/add-repository /path/to/local/repo`

Steps:

1. **Clone** the repo into `repos/` using `git clone <url> repos/<repo-name>`. Infer the repo name from the URL (e.g., `my-app` from `git@github.com:org/my-app.git`). If a directory with that name already exists in `repos/`, inform the user and stop.

2. **Analyze** the newly cloned repo:
   - Read its README.md (or README) if one exists
   - Check its `package.json`, `Cargo.toml`, `go.mod`, `pyproject.toml`, or equivalent to understand the tech stack
   - Read its git remote URL (`git -C repos/<name> remote get-url origin`)
   - Briefly look at the top-level directory structure
   - Check which agent instruction files exist: `CLAUDE.md`, `AGENTS.md`, `.github/copilot-instructions.md`

3. **Update `repos/repos.yaml`** by adding an entry for the new repo:

   ```yaml
   - name: <repo-name>
     url: <git remote URL>
     branch: <current branch>
     prefix: <short prefix, inferred from repo name — e.g., "fe" for frontend, "be" for backend>
     description: <1-line summary>
   ```

   If `repos/repos.yaml` has an empty list (`repos: []`), replace it with the new entry. Otherwise append to the list. Infer a sensible `prefix` from the repo name (first 2-4 characters or a natural abbreviation). If unsure, ask the user.

4. **Update** `repos/repos.md` by appending a new section for the repo in this format:

   ```markdown
   ## <repo-name>

   - **Remote**: <git remote URL>
   - **Tech stack**: <languages, frameworks, key dependencies>
   - **Description**: <1-2 sentence summary from README or inferred from code>
   - **Key paths**: <notable top-level directories and what they contain>
   - **Agent instructions**: <list which of CLAUDE.md, AGENTS.md, .github/copilot-instructions.md exist, or "None">
   ```

   If `repos/repos.md` still has the placeholder comment (`<!-- No repositories found... -->`), remove it before appending.
