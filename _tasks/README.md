# Task System

Task files persist planning context between agent sessions. Each task scopes work to specific repos and embeds the cross-repo context needed to execute it.

## File Naming

Format: `<prefix>-<number>-<slug>.md`

| Prefix | Scope | Example |
|--------|-------|---------|
| Repo prefix (from `repos/repos.yaml`) | Single repo | `fe-1-auth-ui.md`, `be-2-user-api.md` |
| `x` | Cross-cutting | `x-3-auth-flow.md` |

## Task File Format

```markdown
---
status: backlog | active | completed
repos: [frontend, backend-api]
created: 2026-03-30
---

# Task: Feature Name

## Context
What and why. Keep it brief.

## Repo Context
Embedded API surfaces, types, schemas. The task carries its own context.

### frontend
- `src/components/Auth.tsx` — current auth UI
- Exports: `LoginForm`, `useAuth` hook

### backend-api
- `POST /api/auth/login` → `{ token, refreshToken }`
- Type: `AuthResponse { token: string, expires: number }`

## Key Decisions
- Decision 1: chose X because Y

## Important Files
- `repos/frontend/src/auth/LoginForm.tsx` — needs OAuth button
- `repos/backend-api/src/routes/auth.ts` — needs Google endpoint

## Phases

### Phase 1: Backend OAuth
- [ ] Add Google OAuth client config
- [ ] Create `POST /api/auth/google` endpoint
- [ ] Add integration tests

### Phase 2: Frontend OAuth UI
- [ ] Add Google sign-in button
- [ ] Wire OAuth redirect flow
- [ ] Add loading/error states
```

## Commands

- `/create-task` — create with auto-populated context
- `/list-tasks` — filter by status, prefix, or repo
- Hand task file to a scoped worker agent to implement
- Edit `status` in frontmatter as work progresses

## Why

1. **Session continuity** — fresh agents get full context from the task file, no re-explaining
2. **Context distillation** — relevant API surfaces and types embedded in the task, no full codebase reads
