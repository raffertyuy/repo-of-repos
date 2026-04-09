---
name: remove-repository
description: Remove a repo or local folder from repos/, updating repos.yaml, repos.md, and .gitignore
user-invocable: true
---

# Remove Repository

Remove a single repository or local source folder from the workspace. This is the inverse of `/add-repository`.

The user will provide a repo name. For example:
- `/remove-repository frontend` — remove the `frontend` entry
- `/remove-repository backend-api` — remove the `backend-api` entry

## Behavior

1. **Validate** the entry exists in `repos/repos.yaml` by name
   - If not found, report the error and list available entries so the user can pick the right one
   - If found, proceed

2. **Show what will happen** — before taking any action, display:
   - The entry being removed (name, type, URL if git)
   - Whether the directory exists on disk
   - Ask the user: **delete the directory from disk too, or just remove the config entry?**

3. **Remove from `repos/repos.yaml`** — delete the entry from the `repos:` list
   - If this was the last entry, leave `repos: []`

4. **Remove from `repos/repos.md`** — delete the `## <name>` section and all its content up to the next `## ` heading or end of file
   - If no entries remain, restore the placeholder: `<!-- No repositories found. Clone repos into repos/ and run /pull-all-repos to populate this file. -->`

5. **Update `.gitignore`** — if `repos/<name>/` appears in `.gitignore` (whether git repo or gitignored local folder), remove the line. No need to ask — if the entry is being removed, its gitignore line should go too.

6. **Optionally delete from disk** — only if the user confirmed deletion in step 2:
   - Run `rm -rf repos/<name>/`
   - If the user said no, leave the directory in place (it becomes an orphan that `/pull-all-repos` would re-register)

7. **Report** what was done:
   > Removed `<name>` from repos.yaml, repos.md, and .gitignore. Directory on disk: deleted / kept.

## Notes

- Always ask before deleting from disk — never auto-delete
- If the directory has uncommitted changes (for git repos), warn the user before they confirm deletion
- This skill does NOT push or modify anything in the sub-repo's own git history
