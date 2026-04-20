# Plans

Plan files drive the plan-implement workflow. Each plan scopes work to specific repos, embeds context, and uses markdown task lists (`- [ ]`) to track progress.

## Workflow

1. **`/create-plan`** — create a plan with steps, context, and pseudocode
2. **`/implement-plan`** — execute the plan, check off steps, document fixes

## File Naming

Format: `YYYYMMDD-<plan-name>.plan.md`

- Date prefix is today's date (e.g., `20260409`)
- Plan name is 2-4 words, lowercase, hyphen-separated
- Examples: `20260409-google-oauth.plan.md`, `20260410-schema-migration.plan.md`

## Plan File Format

```markdown
---
status: draft | in-progress | completed
repos: [frontend, backend-api]
created: 2026-04-09
---

# Plan: Feature Name

## Context
What and why. Keep it brief.

## Repo Context

### frontend
- `src/components/Auth.tsx` — current auth UI
- Exports: `LoginForm`, `useAuth` hook

### backend-api
- `POST /api/auth/login` -> `{ token, refreshToken }`
- Type: `AuthResponse { token: string, expires: number }`

## Steps

- [ ] Step 1: Backend OAuth endpoint
  - **Task**: Add Google OAuth client and create POST /api/auth/google
  - **Files**: `repos/backend-api/src/routes/auth.ts`, `repos/backend-api/src/config/oauth.ts`
  - **Pseudocode**: Validate Google token, exchange for session, return AuthResponse

- [ ] Step 2: Frontend OAuth UI
  - **Task**: Add Google sign-in button and wire redirect flow
  - **Files**: `repos/frontend/src/components/LoginForm.tsx`
  - **Pseudocode**: Render Google button, handle OAuth callback, store token

- [ ] Step 3: Integration tests
  - **Task**: Add tests for the full OAuth flow
  - **Files**: `repos/backend-api/tests/auth.test.ts`
```

## Status Lifecycle

| Status | Meaning |
|--------|---------|
| `draft` | Plan created, not yet implemented |
| `in-progress` | `/implement-plan` is actively executing steps |
| `completed` | All steps done, tested, verified |

## How /implement-plan Updates the Plan

When `/implement-plan` runs against a plan:

1. Sets `status: in-progress`
2. Works through each unchecked step sequentially
3. Marks `- [ ]` to `- [x]` as each step completes
4. Adds `**Notes**` under steps if anything unexpected is discovered
5. Runs the app and tests after implementation
6. Documents any fixes as new checked-off steps
7. Sets `status: completed` when all steps pass

The plan becomes a living document — a record of what was planned AND what actually happened.

## Why

1. **Session continuity** — fresh agents get full context from the plan, no re-explaining
2. **Trackable progress** — checkboxes show exactly where implementation stands
3. **Self-documenting** — the plan records decisions, discoveries, and fixes as they happen
