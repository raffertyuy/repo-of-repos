---
name: clean-repos
description: Find and remove stale entries from repos.yaml whose directories no longer exist on disk
user-invocable: true
origin: template
---

# Clean Repos

Scan `repos/repos.yaml` for stale entries — entries whose corresponding `repos/<name>/` directory no longer exists on disk — and remove them from the configuration.

## Order of Operations

1. **Parse `repos/repos.yaml`** — read the `repos:` list
   - If the list is empty, report "Nothing to clean" and stop

2. **Check each entry** — for every entry, check if `repos/<name>/` exists on disk

3. **Collect stale entries** — entries where the directory does NOT exist
   - If no stale entries found, report "All entries are present on disk. Nothing to clean." and stop

4. **Show the stale entries** — display a table:

   | Entry | Type | URL | Status |
   |-------|------|-----|--------|
   | `<name>` | git | `<url>` | Directory missing |
   | `<name>` | local | — | Directory missing |

5. **Ask for confirmation** — "Remove these N entries from the config? (yes/no)"
   - If the user says no, stop without changes

6. **Remove stale entries from `repos/repos.yaml`** — delete each stale entry from the `repos:` list
   - If all entries were stale, leave `repos: []`

7. **Remove stale entries from `repos/repos.md`** — for each stale entry, delete its `## <name>` section and all content up to the next `## ` heading or end of file
   - If no entries remain, restore the placeholder: `<!-- No repositories found. Clone repos into repos/ and run /pull-all-repos to populate this file. -->`

8. **Update `.gitignore`** — for each stale entry, if `repos/<name>/` appears in `.gitignore` (whether git repo or gitignored local folder), remove the line. No need to ask — stale entries shouldn't leave gitignore debris.

9. **Report** a summary:

   | Entry | Type | Action |
   |-------|------|--------|
   | `<name>` | git | Removed from repos.yaml, repos.md, .gitignore |
   | `<name>` | local | Removed from repos.yaml, repos.md (and .gitignore if present) |

## Notes

- This skill never deletes directories — it only cleans up config for directories that are already gone
- The inverse scenario (directories exist but aren't in config) is handled by `/pull-all-repos` orphan detection
- Always confirm before making changes — never auto-remove
- Safe to run as a dry-run check: if there are no stale entries, it just reports "nothing to clean"
