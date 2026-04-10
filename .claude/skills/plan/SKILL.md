---
name: plan
description: Create an implementation plan in _plans/ with embedded repo context and step-by-step task lists
user-invocable: true
origin: template
---

# Plan

Create a detailed implementation plan before any coding begins. The plan is a markdown file with checkboxes that `/implement` will later execute and update.

## Arguments

The user provides a description of the work to be done. For example:
- `/plan Add Google OAuth to the login flow`
- `/plan Migrate user table to new schema`
- `/plan Update API rate limiting across all services`

## Steps

### 1. Determine Scope

Based on the user's description, determine which repos are involved:

1. Read `repos/repos.yaml` to get the list of repos/folders
2. Identify which repos are relevant to the work
3. Ask the user to clarify scope only if it's genuinely ambiguous

### 2. Generate the Filename

Format: `YYYYMMDD-<plan-name>.plan.md`

- Use today's date as the prefix (e.g., `20260409`)
- Create a short, descriptive plan name from the description:
  - Lowercase, hyphen-separated
  - 2-4 words maximum
  - Example: "Add Google OAuth to login" -> `20260409-google-oauth.plan.md`

### 3. Gather Context

Use the explorer agent pattern (read-only) to gather relevant context from the repos in scope:

1. Read `repos/repos.md` for an overview
2. For each repo in scope, identify and read:
   - API endpoints or routes relevant to the work
   - Type definitions, interfaces, or schemas involved
   - Key files that will need changes
   - Any existing tests for the affected code
3. Distill this into compact summaries — don't paste entire files

### 4. Write the Plan File

Create `_plans/YYYYMMDD-<plan-name>.plan.md` with this structure:

```markdown
---
status: draft
repos: [<repo-names>]
created: <today's date>
---

# Plan: <descriptive title>

## Context
<1-3 sentences: what needs to happen and why>

## Repo Context
<embedded API surfaces, types, schemas from relevant repos>

### <repo-name>
- <key files with brief descriptions>
- <exported types or endpoints relevant to this work>
- <architectural constraints or patterns to follow>

(repeat for each repo in scope)

## Steps

- [ ] Step 1: <brief title>
  - **Task**: <detailed explanation of what to do>
  - **Files**: <list of files to create or modify, with full paths>
  - **Pseudocode**: <high-level pseudocode, NOT real code>

- [ ] Step 2: <brief title>
  - **Task**: <detailed explanation>
  - **Files**: <file list>
  - **Pseudocode**: <pseudocode>

(continue for all steps)

- [ ] Step N: Validate
  - **Task**: Run the application and tests, verify everything works
  - **Files**: <test files>
```

### 5. Plan Quality Checklist

Before finishing, verify the plan:
- [ ] Steps are ordered by dependency (do X before Y if Y depends on X)
- [ ] Each step is small enough to implement and verify independently
- [ ] Pseudocode is used, not real code — implementation details are for `/implement`
- [ ] Files listed actually exist (or are clearly marked as "new file")
- [ ] A validation step exists at the end
- [ ] Any user intervention points are called out

### 6. Report

Tell the user:
- The plan file path
- A brief summary of the steps
- Which repos are in scope
- Suggested next step: "Run `/implement _plans/<filename>` when ready to build."

## Notes

- Always read from repos to gather real context — don't fabricate API surfaces or types
- If a repo in scope has no code yet, note that in the context section
- Keep it simple — avoid over-architecture. The plan should be the simplest path to done.
- Use pseudocode only, not real code
- Call out any points where user input or decisions are needed
- Plans are tracked in git — they're workspace artifacts
- Review the plan with the user before they run `/implement`
