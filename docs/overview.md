# Project Overview & Architecture

## Project Overview

### What the System Does

FlowForge is a full-stack agile project management application designed for small teams of 3–10 users. It provides structured work tracking across a three-level hierarchy and exposes a REST API consumed by a React single-page application.

### Target Users

Small engineering or product teams that need lightweight agile tooling without the overhead of enterprise tools. Each user has a global role (`admin`, `member`, or `viewer`) and a separate per-project role, allowing fine-grained access control across multiple projects.

### Core Hierarchy

```
Project
  └── User Story
        └── Task
              ├── Comment
              └── Attachment
```

- A **Project** is the top-level container. It has an owner, a set of members with roles, a status, and optional date/hour tracking.
- A **User Story** belongs to exactly one project. It carries priority, story points, business value, acceptance criteria, and a status.
- A **Task** belongs to exactly one story. It supports multiple assignees (M2M), a due date, estimated/actual hours, labels, task type, and file attachments.

### Key Capabilities

- Full CRUD at every level of the hierarchy (Project, Story, Task)
- Role-based access control: `admin`, `member`, `viewer` at both global and per-project scope
- Kanban board with drag-and-drop status transitions (frontend, powered by dnd-kit)
- In-app notification system with unread count polling
- Background job system: deadline reminders, notification cleanup, on-demand project reports
- Project analytics: task/story counts, completion rate, hour tracking, member count
- CSV and JSON project export
- File attachments on tasks (images, PDFs, documents — max 5 MB)
- Activity log per project tracking every mutation
- Avatar upload for user profiles
- Admin panel: user management (role changes, activate/deactivate), job monitoring
- Global search across projects, stories, and tasks (by title/description) with filters for status, priority, assignee, and project
- Dark mode with system preference detection, persisted in `localStorage`

---

## Architecture Notes

### High-Level Architecture

```
┌──────────────────────────┐        HTTP / JSON         ┌────────────────────────────┐
│     React SPA            │  ─────────────────────►   │     Django REST API        │
│     Vite  :5173          │  ◄─────────────────────   │     DRF  :8000             │
│                          │    Bearer <access_token>   │                            │
│  TanStack Query (cache)  │                            │  JWT via SimpleJWT         │
│  Zustand (UI state)      │                            │  DRF serializers           │
│  Axios (HTTP + refresh)  │                            │  Custom permission classes │
└──────────────────────────┘                            └────────────┬───────────────┘
                                                                     │
                                                          ┌──────────▼──────────┐
                                                          │     SQLite DB       │
                                                          │   (db.sqlite3)      │
                                                          └──────────┬──────────┘
                                                                     │
                                                          ┌──────────▼──────────┐
                                                          │   APScheduler       │
                                                          │  (in-process jobs)  │
                                                          └─────────────────────┘
```

There is no server-side rendering. The React frontend is a fully decoupled SPA that communicates with the Django backend exclusively over HTTP.

### API Layer

The backend is structured as a Django project (`flowforge/`) with eight Django apps under `apps/`:

| App | Responsibility |
|-----|---------------|
| `users` | Custom user model, JWT auth, profile, avatar upload |
| `projects` | Project CRUD, members, invitations, analytics, CSV/JSON export |
| `stories` | User story CRUD |
| `tasks` | Task CRUD, status transitions, assignment, comments, attachments |
| `notifications` | In-app notification CRUD and unread count |
| `jobs` | Background job model, scheduler, task execution with retry |
| `teams` | Team and team invitation management |
| `core` | Shared utilities: pagination, permission classes, validators, sanitization |

All API views use DRF class-based views (`APIView`, `generics.*`). Serializers handle all input validation. Custom permission classes (`IsProjectMember`, `IsProjectAdmin`) enforce object-level access control.

### Data Flow (Example: Creating a Task)

```
1. Frontend sends POST /api/stories/:id/tasks/  with Bearer token
2. JWTAuthentication middleware validates the access token
3. TaskListCreateView.create() checks ProjectMember role (admin or member required)
4. TaskCreateSerializer validates all fields
5. Task is saved; assigned_to M2M is set
6. log_activity() writes an ActivityLog entry
7. TaskSerializer serializes the new task and returns HTTP 201
```

### Async / Background Workflow

Background jobs run via **APScheduler** inside the Django process. The scheduler is started via `apps/jobs/scheduler.py` and registers two recurring jobs:

| Job | Trigger | What it does |
|-----|---------|-------------|
| `deadline_reminder` | Every hour | Queries tasks with `due_date = tomorrow` and status `todo`/`in_progress`. Creates one `Notification` per assigned user. Deduplicates — skips if a reminder was already sent today for the same task/user pair. |
| `notification_cleanup` | Daily at 03:00 | Deletes `Notification` rows where `is_read=True` and `created_at < now - 30 days`. |

A third job, `project_report`, is triggered on demand via `POST /api/projects/:id/report/`. It runs in a daemon thread, collects story/task statistics, and stores a JSON summary as the `BackgroundJob.result` field.

All three jobs are wrapped in `run_job_with_tracking()` which:
1. Creates a `BackgroundJob` row with `status=running`
2. Executes the job function
3. On success: sets `status=completed`, stores result, records `executed_at`
4. On failure: increments `retry_count`, stores `error_message`, applies exponential backoff (2s → 4s → 8s), retries up to `max_retries` (default 3)
5. After max retries: sets `status=failed`, logs permanently

Failed jobs never crash the scheduler — exceptions are caught and logged. The `background_jobs` table is a full audit trail of every execution.

### Why This Architecture Fits Small Teams

- **SQLite** removes the need for a separate database server. For 3–10 concurrent users with moderate write volume, SQLite is sufficient.
- **APScheduler in-process** avoids the Redis + Celery broker infrastructure that would be required for a distributed task queue. The tradeoff (jobs share web server resources, don't survive restarts) is acceptable at this scale.
- **Polling over WebSockets** — notifications poll every 30 seconds, Kanban refetches every 60 seconds. Django Channels would require Redis. For a small team, polling latency is acceptable.
- **Decoupled SPA** — the React frontend can be deployed to a CDN independently of the Django backend, and the same API can serve a mobile client in the future.
- **TanStack Query + Zustand split** — server state (projects, tasks, stories) is managed by TanStack Query for caching and background refetch. Global UI state (auth, theme, notifications) is in Zustand. This avoids manually managing async state and keeps stores lean.
