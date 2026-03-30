---
name: list-tasks
description: List tasks from _tasks/ filtered by status, prefix, or repo
user-invocable: true
---

# List Tasks

Display tasks from the `_tasks/` directory, optionally filtered by status, prefix, or repo.

## Arguments

The user may provide optional filters:
- `/list-tasks` — show all tasks
- `/list-tasks active` — filter by status
- `/list-tasks fe` — filter by prefix
- `/list-tasks frontend` — filter by repo name
- `/list-tasks active fe` — combine filters

## Steps

1. **Read all task files** in `_tasks/` (exclude `README.md`)
2. **Parse frontmatter** from each file to extract `status`, `repos`, and `created` fields
3. **Extract the prefix** from the filename (e.g., `fe` from `fe-1-auth-ui.md`)
4. **Apply filters** if the user provided any:
   - Status filter: match the `status` field (`backlog`, `active`, `completed`)
   - Prefix filter: match the filename prefix
   - Repo filter: match any repo in the `repos` list (check both repo name and prefix via `repos/repos.yaml`)
5. **Display** the results as a table

## Output Format

```
## Tasks

| # | Task | Status | Repos | Created |
|---|------|--------|-------|---------|
| fe-1 | Auth UI redesign | active | frontend | 2026-03-28 |
| be-2 | User API migration | backlog | backend-api | 2026-03-29 |
| x-3 | Auth flow update | active | frontend, backend-api | 2026-03-30 |

Active: 2 | Backlog: 1 | Completed: 0
```

## Notes

- If `_tasks/` is empty or only contains `README.md`, inform the user and suggest `/create-task`
- Sort by status (active first, then backlog, then completed), then by number
- Keep the output concise — show the table, not the full task contents
- If the user wants task details, they can ask to read the specific task file
