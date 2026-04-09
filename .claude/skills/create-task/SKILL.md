---
name: create-task
description: Create a task file in _tasks/ with prefix routing and embedded repo context
user-invocable: true
---

# Create Task

Create a new task file in `_tasks/` with prefix-based routing and embedded context from relevant repos.

## Arguments

The user provides a description of the work to be done. For example:
- `/create-task Add Google OAuth to the login flow`
- `/create-task Migrate user table to new schema`
- `/create-task Update API rate limiting across all services`

## Steps

### 1. Determine Scope

Based on the user's description, determine which repos are involved:

1. Read `repos/repos.yaml` to get the list of repos/folders and their prefixes (both `type: git` and `type: local` entries are valid scopes)
2. If the work is clearly scoped to one repo or folder, use that entry's prefix
3. If the work spans multiple repos/folders, use the `x` (cross-cutting) prefix
4. Ask the user to clarify scope only if it's genuinely ambiguous

### 2. Assign a Number

Look at existing files in `_tasks/` to find the next available number. Numbers are global (not per-prefix):
- If `_tasks/` has `fe-1-auth-ui.md` and `x-2-schema-update.md`, the next number is `3`
- If `_tasks/` is empty (only README.md), start at `1`

### 3. Generate a Slug

Create a short, descriptive slug from the task description:
- Lowercase, hyphen-separated
- 2-4 words maximum
- Example: "Add Google OAuth to login" → `google-oauth`

### 4. Gather Context

Use the explorer agent pattern (read-only) to gather relevant context from the repos in scope:

1. Read `repos/repos.md` for an overview
2. For each repo in scope, identify and read:
   - API endpoints or routes relevant to the task
   - Type definitions, interfaces, or schemas involved
   - Key files that will need changes
   - Any existing tests for the affected code
3. Distill this into compact summaries — don't paste entire files

### 5. Write the Task File

Create `_tasks/<prefix>-<number>-<slug>.md` with this structure:

```markdown
---
status: backlog
repos: [<repo-names>]
created: <today's date>
---

# Task: <descriptive title>

## Context
<1-3 sentences: what needs to happen and why>

## Repo Context
<embedded API surfaces, types, schemas from relevant repos>

### <repo-name>
- <key files with brief descriptions>
- <exported types or endpoints relevant to this task>
- <architectural constraints or patterns to follow>

(repeat for each repo in scope)

## Key Decisions
<leave empty or pre-fill if the user specified constraints>

## Important Files
<list of files that will likely need changes, with full paths>

## Phases
<break the work into phases, one per repo or logical unit>

### Phase 1: <description>
- [ ] <specific task item>
- [ ] <specific task item>
```

### 6. Report

Tell the user:
- The task file path
- A brief summary of what was captured
- Which repos are in scope
- Suggested next step (e.g., "hand this to a worker agent scoped to `repos/backend-api/`")

## Notes

- Always read from repos to gather real context — don't fabricate API surfaces or types
- If a repo in scope has no code yet (empty or just scaffolded), note that in the context section
- Keep the context section compact — this is a summary, not a code dump
- Task files are tracked in git (not gitignored) — they're workspace artifacts
