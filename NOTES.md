# Implementation Notes

## Requirements Checklist

### Auth
- [x] POST `/auth/register` — name, email, password
- [x] POST `/auth/login` — returns JWT
- [x] Passwords hashed with bcrypt (cost 12)
- [x] JWT expiry 24h with user_id and email claims
- [x] All non-auth endpoints require Bearer token

### Projects API
- [x] GET `/projects` — lists projects user owns or has tasks in
- [x] POST `/projects` — creates project, owner = current user
- [x] GET `/projects/:id` — project details + tasks
- [x] PATCH `/projects/:id` — owner only
- [x] DELETE `/projects/:id` — owner only, cascades tasks

### Tasks API
- [x] GET `/projects/:id/tasks` — with `?status=`, `?priority=`, and `?assignee=` filters
- [x] POST `/projects/:id/tasks` — create task
- [x] PATCH `/tasks/:id` — update any field
- [x] DELETE `/tasks/:id` — project owner or task creator only

### General API
- [x] JSON responses everywhere
- [x] 400 validation errors with `{"error": "validation failed", "fields": {...}}`
- [x] 401 vs 403 distinction
- [x] 404 with `{"error": "not found"}`
- [x] Structured logging with slog
- [x] Graceful shutdown on SIGTERM/SIGINT
- [x] Request logging middleware (method, path, status, duration, IP)

### Frontend
- [x] Login page with client-side validation and error handling
- [x] Register page with validation
- [x] Projects list with create button
- [x] Project detail with kanban columns, filters (status, priority, assignee)
- [x] Task create/edit modal with title, status, priority, assignee search, due date
- [x] Navbar with user name, dark mode toggle, and logout
- [x] React Router navigation
- [x] Auth persists across refresh (localStorage)
- [x] Protected routes redirect to /login
- [x] Loading and error states on every page
- [x] Optimistic UI for task updates (revert on error)
- [x] Responsive at 375px and 1280px
- [x] Empty states everywhere (no blank screens)
- [x] Custom Tailwind components (no heavy UI library)

### Infrastructure
- [x] docker-compose.yml at root — PostgreSQL + backend + frontend
- [x] Single `docker compose up` with zero manual steps
- [x] PostgreSQL credentials via .env
- [x] .env.example with all variables
- [x] Multi-stage Dockerfiles (both backend and frontend)
- [x] golang-migrate for migrations
- [x] Migrations run automatically on startup
- [x] Up and down migration files
- [x] Seed script — 2 users, 3 projects, 16 tasks across all statuses
- [x] Database resets on every build (tmpfs, fresh migrations + seed each start)

### README
- [x] Overview with tech stack
- [x] Architecture decisions
- [x] Running locally instructions
- [x] Migration docs
- [x] Test credentials
- [x] API reference with examples
- [x] "What I'd do with more time" section

---

## What We Did Beyond the Requirements

### Bonus features (from rubric)
- **Pagination** on both `/projects` and `/projects/:id/tasks` with `?page=` and `?limit=` query params
- **GET `/projects/:id/stats`** — task counts grouped by status and by assignee
- **Drag-and-drop** kanban board — drag tasks between To Do / In Progress / Done columns to update status, powered by @dnd-kit with optimistic updates
- **Dark mode** toggle that persists across sessions (localStorage), respects system preference on first visit

### Extra endpoints
- **GET `/projects/:id/members`** — returns users involved in a project (owner + task assignees), powers the assignee filter and dropdown
- **GET `/users/search?q=`** — search users by name or email for task assignment
- **Priority filter** — backend supports `?priority=` filter on task list (spec only asked for status and assignee)

### Architecture extras
- **Hexagonal architecture** (ports & adapters) — domain layer has zero external deps, repository interfaces in domain, implementations in infra/postgres
- **Global error handler middleware** — all errors flow through a single middleware that formats AppError/ServerError/ValidationError consistently
- **Request logging middleware** — logs every request with method, path, status, duration, IP via slog
- **URL-based filters** — filters use `useSearchParams` so filtered views are shareable via URL

### Frontend extras
- **Kanban board with drag-and-drop** — tasks grouped by status in three columns, drag between columns to change status
- **User search for assignees** — type-ahead search input in task form to find and assign any registered user
- **Custom confirm dialogs** — no browser `confirm()`, custom modal with warning icon, cancel/confirm buttons, loading state
- **Dark mode** — full dark theme across all components, toggle in navbar, persists to localStorage
- **Overdue date highlighting** — tasks past due date show red date text
- **Assignee names on cards** — task cards show actual user names instead of generic "Assigned"
- **Multi-stage frontend Dockerfile** — builds with Node, serves with nginx (SPA fallback configured)
- **Makefile** — `make up`, `make seed`, `make clean`, `make logs` shortcuts

---

## Not Implemented

- **Integration tests** — no `*_test.go` files. Would use testcontainers with a real Postgres instance.
- **Real-time updates** — no WebSocket or SSE. Uses React Query cache invalidation instead.
- **E2E tests** — would add Playwright covering login → create project → add task → drag to done flow.
