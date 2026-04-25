# ⚡ FlowForge

### Agile Project Management for Small Teams

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat&logo=python&logoColor=white)
![Django](https://img.shields.io/badge/Django-4.2-092E20?style=flat&logo=django&logoColor=white)
![React](https://img.shields.io/badge/React-19-61DAFB?style=flat&logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-5.0-3178C6?style=flat&logo=typescript&logoColor=white)
![TailwindCSS](https://img.shields.io/badge/Tailwind-v4-06B6D4?style=flat&logo=tailwindcss&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-003B57?style=flat&logo=sqlite&logoColor=white)

A full-stack agile project management tool built for small teams (3–10 users).  
Hierarchical work tracking · Role-based access · Kanban board · Background jobs · In-app notifications

[![▶ Watch Demo](https://img.shields.io/badge/▶%20Watch_Demo-VEED.IO-6B4FBB?style=for-the-badge&logo=vimeo&logoColor=white)](https://www.veed.io/view/87c4417e-e120-4cd6-8941-cd07f2557acf?source=editor&panel=share)

🎬 **Demo:** https://www.veed.io/view/87c4417e-e120-4cd6-8941-cd07f2557acf?source=editor&panel=share

---

## 📋 Table of Contents

1. [Project Overview](#1--project-overview)
2. [Tech Stack](#2--tech-stack)
3. [Architecture](#3--architecture)
4. [Setup Instructions](#4--setup-instructions)
5. [API Documentation](#5--api-documentation)
6. [Database Schema](#6--database-schema)
7. [Async / Background Workflow](#7--async--background-workflow)
8. [Design Decisions & Tradeoffs](#8--design-decisions--tradeoffs)
9. [Security Considerations](#9--security-considerations)
10. [AI Usage](#10--ai-usage)
11. [What I'd Improve or Build Next](#11--what-id-improve-or-build-next)

---

## 1. 🎯 Project Overview

FlowForge models the standard agile hierarchy — every piece of work lives in exactly one place:

```
📁 Project
   └── 📖 User Story
         └── ✅ Task
               ├── 💬 Comment
               └── 📎 Attachment
```

### Core Capabilities

| Feature | Details |
|---------|---------|
| 🗂️ Hierarchy | Full CRUD at Project → Story → Task level |
| 🔐 Role-Based Access | `admin` · `member` · `viewer` at global and per-project scope |
| 🗃️ Kanban Board | Drag-and-drop status transitions powered by dnd-kit |
| 🔔 Notifications | In-app notifications with unread badge, polled every 30s |
| ⚙️ Background Jobs | Deadline reminders · Notification cleanup · On-demand reports |
| 📊 Analytics | Completion rate, hour tracking, story points, member stats |
| 📤 Export | CSV and JSON project export |
| 📎 File Attachments | Images, PDFs, documents — max 5 MB per file |
| 🔍 Global Search | Search across projects, stories, and tasks simultaneously |
| 🌙 Dark Mode | System-aware, persisted in `localStorage` |
| 🛡️ Admin Panel | User management, role changes, job monitoring |

---

## 2. 🛠️ Tech Stack

### Frontend

| Layer | Technology |
|-------|-----------|
| Framework | React 19 + TypeScript + Vite |
| Styling | Tailwind CSS v4 |
| Server State | TanStack Query v5 |
| UI State | Zustand v5 |
| Drag & Drop | dnd-kit |
| Forms | React Hook Form + Zod |
| HTTP Client | Axios with JWT interceptors |
| Charts | Recharts |
| Animations | Framer Motion |

### Backend

| Layer | Technology |
|-------|-----------|
| Framework | Django 4.2 + Django REST Framework |
| Authentication | SimpleJWT (access + refresh tokens) |
| Database | SQLite |
| Background Jobs | APScheduler + custom job tracker |
| API Docs | drf-spectacular (Swagger UI + ReDoc) |
| File Uploads | Django media files |
| HTML Sanitization | bleach |
| PDF Reports | ReportLab |

---

## 3. 🏗️ Architecture

FlowForge is a fully decoupled frontend/backend application. The React SPA communicates with the Django REST API exclusively over HTTP. There is no server-side rendering.

```
┌──────────────────────┐         REST / JSON          ┌───────────────────────┐
│   React Frontend     │  ──────────────────────────► │   Django Backend      │
│   Vite  :5173        │  ◄────────────────────────── │   DRF  :8000          │
└──────────────────────┘     Bearer <access_token>     └──────────┬────────────┘
                                                                   │
                                                        ┌──────────▼────────────┐
                                                        │      SQLite DB        │
                                                        └──────────┬────────────┘
                                                                   │
                                                        ┌──────────▼────────────┐
                                                        │  APScheduler          │
                                                        │  (in-process jobs)    │
                                                        └───────────────────────┘
```

### Backend App Structure

```
backend/
├── flowforge/          # Django project — settings, urls, wsgi
└── apps/
    ├── users/          # Custom user model, JWT auth, profile, avatar upload
    ├── projects/       # Project CRUD, members, invitations, analytics, export
    ├── stories/        # User stories
    ├── tasks/          # Tasks, comments, attachments, activity logs
    ├── notifications/  # In-app notifications
    ├── jobs/           # Background job tracking and execution
    ├── teams/          # Teams and team invitations
    └── core/           # Shared: pagination, permissions, search, validators
```

### Frontend Structure

```
frontend/src/
├── api/            # Axios API layer — all HTTP calls, JWT interceptors
├── components/     # Kanban board, modals, layout, common UI
├── hooks/          # TanStack Query hooks (useProjects, useTasks, …)
├── pages/          # Route-level components
├── store/          # Zustand stores (auth, notifications, theme)
└── types/          # Shared TypeScript interfaces
```

### Pages & Routes

| Route | Description |
|-------|-------------|
| `/login` | JWT login |
| `/register` | Account creation |
| `/dashboard` | Overview — projects, activity feed, metrics |
| `/projects` | Project list with search and archive toggle |
| `/projects/:id` | Kanban board with story panel and filters |
| `/stories/:id` | Story detail — tasks, comments, attachments |
| `/profile` | Avatar upload, profile edit, password change |
| `/admin` | User management, job monitor (admin only) |

---

## 4. 🚀 Setup Instructions

### Prerequisites

| Tool | Version |
|------|---------|
| Python | 3.10+ |
| Node.js | 18+ |
| npm | 9+ |

### ① Backend

```bash
cd backend

# Create and activate virtual environment
python -m venv venv
venv\Scripts\activate          # Windows
source venv/bin/activate       # macOS / Linux

# Install dependencies
pip install -r requirements.txt

# Create backend/.env
SECRET_KEY=your-secret-key-here
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1
CORS_ALLOWED_ORIGINS=http://localhost:5173,http://127.0.0.1:5173

# Apply migrations and start
python manage.py migrate
python manage.py runserver
# → http://localhost:8000
```

### ② Frontend

```bash
cd frontend

# Install dependencies
npm install

# Create frontend/.env
VITE_API_BASE_URL=http://localhost:8000

# Start dev server
npm run dev
# → http://localhost:5173
```

### ③ Auto-generated API Docs

> Start the backend first, then open any of these:

| URL | Description |
|-----|-------------|
| http://localhost:8000/api/docs/ | Swagger UI |
| http://localhost:8000/api/redoc/ | ReDoc |
| http://localhost:8000/api/schema/ | OpenAPI JSON schema |

---

## 5. 📡 API Documentation

**Base URL:** `http://localhost:8000/api`  
**Auth:** All endpoints require `Authorization: Bearer <access_token>` except `/auth/register/` and `/auth/login/`.

### Authentication

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/auth/register/` | Register new user |
| `POST` | `/auth/login/` | Login — returns access + refresh tokens |
| `POST` | `/auth/refresh/` | Refresh access token |
| `GET/PATCH` | `/auth/profile/` | Get or update current user profile |
| `POST` | `/auth/profile/avatar/` | Upload avatar image |
| `POST` | `/auth/profile/password/` | Change password |

### Projects

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET/POST` | `/projects/` | List or create projects |
| `GET/PATCH/DELETE` | `/projects/:id/` | Get, update, or delete project |
| `POST` | `/projects/:id/archive/` | Archive / unarchive |
| `GET/POST` | `/projects/:id/members/` | List members or add member |
| `DELETE` | `/projects/:id/members/:user_id/` | Remove member |
| `PATCH` | `/projects/:id/members/:user_id/role/` | Update member role |
| `POST` | `/projects/:id/invite/` | Invite user (status: pending) |
| `POST` | `/projects/invites/:member_id/accept/` | Accept project invite |
| `GET` | `/projects/:id/activity/` | Activity log |
| `GET` | `/projects/:id/analytics/` | Analytics summary |
| `GET` | `/projects/:id/export/` | Export as CSV or JSON |
| `POST` | `/projects/:id/report/` | Trigger async report — returns `202` |

### Stories & Tasks

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET/POST` | `/projects/:project_id/stories/` | List or create stories |
| `GET/PATCH/DELETE` | `/stories/:id/` | Get, update, or delete story |
| `GET/POST` | `/stories/:story_id/tasks/` | List or create tasks |
| `GET/PATCH/DELETE` | `/tasks/:id/` | Get, update, or delete task |
| `PATCH` | `/tasks/:id/status/` | Update task status |
| `POST` | `/tasks/:id/assign/` | Assign users to task |
| `GET/POST` | `/tasks/:task_id/comments/` | List or add comments |
| `DELETE` | `/comments/:id/` | Delete own comment |
| `GET/POST` | `/tasks/:task_id/attachments/` | List or upload attachments |

### Other Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/notifications/` | List notifications |
| `PATCH` | `/notifications/:id/` | Mark as read |
| `POST` | `/notifications/mark-all-read/` | Mark all as read |
| `GET` | `/notifications/unread-count/` | Unread count |
| `GET` | `/search/?q=` | Global search — projects, stories, tasks |
| `GET/POST` | `/teams/` | List or create teams |
| `POST` | `/teams/:id/invite/` | Invite user to team |
| `POST` | `/invitations/:id/accept/` | Accept team invitation |
| `GET` | `/jobs/` | List background jobs |
| `POST` | `/jobs/trigger/` | Manually trigger a job |

---

## 6. 🗄️ Database Schema

### Entity Relationships

```
CustomUser ──< ProjectMember >── Project ──< UserStory ──< Task
                                                              ├──< Comment
                                                              └──< TaskAttachment
Project ──< ActivityLog
Project ──< Notification >── CustomUser
Team ──< TeamInvitation
BackgroundJob  (standalone)
```

### Key Tables

**`users`**

| Column | Type | Notes |
|--------|------|-------|
| `id` | INTEGER PK | |
| `username` | VARCHAR | unique |
| `email` | VARCHAR | unique |
| `full_name` | VARCHAR | |
| `avatar_url` | VARCHAR | path to uploaded image |
| `role` | VARCHAR | `admin` · `member` · `viewer` |
| `created_at` | DATETIME | auto |

**`projects`**

| Column | Type | Notes |
|--------|------|-------|
| `id` | INTEGER PK | |
| `name` | VARCHAR | |
| `owner_id` | FK → users | CASCADE |
| `status` | VARCHAR | `planning` · `active` · `on_hold` · `completed` · `cancelled` |
| `is_archived` | BOOLEAN | default `False` |
| `start_date` / `end_date` | DATE | nullable |
| `estimated_hours` / `actual_hours` | INTEGER | nullable |
| `created_at` / `updated_at` | DATETIME | auto |

**`project_members`**

| Column | Type | Notes |
|--------|------|-------|
| `project_id` | FK → projects | |
| `user_id` | FK → users | |
| `role` | VARCHAR | `admin` · `member` · `viewer` |
| `status` | VARCHAR | `pending` · `accepted` |

Unique constraint: `(project_id, user_id)`

**`user_stories`**

| Column | Type | Notes |
|--------|------|-------|
| `id` | INTEGER PK | |
| `project_id` | FK → projects | |
| `title` | VARCHAR | |
| `status` | VARCHAR | `todo` · `in_progress` · `done` · `blocked` · `testing` |
| `priority` | VARCHAR | `low` · `medium` · `high` · `critical` |
| `story_points` | INTEGER | nullable, 0–100 |
| `acceptance_criteria` | TEXT | |
| `created_by_id` | FK → users | SET NULL |

**`tasks`**

| Column | Type | Notes |
|--------|------|-------|
| `id` | INTEGER PK | |
| `story_id` | FK → user_stories | |
| `title` | VARCHAR | |
| `status` | VARCHAR | `todo` · `in_progress` · `done` · `blocked` · `testing` |
| `priority` | VARCHAR | `low` · `medium` · `high` · `critical` |
| `task_type` | VARCHAR | `feature` · `bug` · `enhancement` · `research` · `testing` |
| `due_date` | DATE | nullable |
| `estimated_hours` / `actual_hours` | DECIMAL(5,2) | nullable |
| `labels` | JSON | list of strings |
| `assigned_to` | M2M → users | |
| `created_by_id` | FK → users | SET NULL |

**`background_jobs`**

| Column | Type | Notes |
|--------|------|-------|
| `id` | INTEGER PK | |
| `job_type` | VARCHAR | `deadline_reminder` · `notification_cleanup` · `project_report` |
| `status` | VARCHAR | `pending` · `running` · `completed` · `failed` |
| `result` | TEXT | JSON result on success |
| `retry_count` / `max_retries` | INTEGER | default 0 / 3 |
| `error_message` | TEXT | nullable |
| `executed_at` | DATETIME | nullable |

> Other tables: `comments` · `task_attachments` · `notifications` · `activity_logs` · `teams` · `team_invitations`

---

## 7. ⚙️ Async / Background Workflow

Background jobs run via **APScheduler** inside the Django process. Every execution is tracked in the `background_jobs` table with full status, result, and error history.

### Scheduled Jobs

| Job | Schedule | What it does |
|-----|----------|-------------|
| `deadline_reminder` | Every hour | Finds tasks due in 24h, creates one notification per assigned user. Deduplicates — one reminder per user per task per day. |
| `notification_cleanup` | Daily 03:00 | Deletes read notifications older than 30 days. |
| `project_report` | On demand | Triggered via `POST /projects/:id/report/`. Generates a JSON stats summary stored as the job result. |

### Retry Logic

```
Attempt 1 fails  →  wait 2s  →  retry
Attempt 2 fails  →  wait 4s  →  retry
Attempt 3 fails  →  wait 8s  →  retry
Attempt 4 fails  →  mark FAILED permanently, log error
```

- `retry_count` and `error_message` persisted to DB on every failure
- Failed jobs never crash the scheduler — other jobs continue running
- The `background_jobs` table is a full audit trail of every execution
- All jobs wrapped in `run_job_with_tracking()` which catches all exceptions

---

## 8. 🧠 Design Decisions & Tradeoffs

| Decision | Choice | Tradeoff |
|----------|--------|----------|
| **Database** | SQLite | No separate server needed for 3–10 users. Not suitable for high concurrent writes at scale. |
| **Background Jobs** | APScheduler (in-process) | No Redis/broker needed. Jobs don't survive restarts. Celery would be the production choice. |
| **Real-time** | Polling (30s notifications, 60s board) | No WebSocket infrastructure needed. Not truly real-time — up to 30s delay. |
| **Token Storage** | Both tokens in `localStorage` | Simple and persistent. Both tokens are XSS-accessible. Production fix: `HttpOnly` cookie. |
| **State Management** | TanStack Query + Zustand split | TanStack Query owns server state; Zustand owns UI state. Clean separation, small learning curve. |
| **Kanban Updates** | Optimistic updates | Instant visual feedback on drag-and-drop. Reverts on API failure with toast error. |
| **Hierarchy** | Strict 3-level (Project → Story → Task) | Simple queries and permissions. No sub-tasks or cross-project stories. |
| **Roles** | Global role + per-project role | Workspace admins vs project admins are separate. Two role fields to check in permission logic. |

---

## 9. 🔒 Security Considerations

### Authentication & Authorization

- **JWT via SimpleJWT** — access tokens expire in **1 hour**, refresh tokens in **7 days** with rotation enabled
- **Three custom permission classes** enforce object-level access:
  - `IsProjectMember` — any membership record grants read access
  - `IsProjectMemberRole` — `admin` or `member` role required for writes
  - `IsProjectAdmin` — `admin` role required for member management and deletion
- Users can only access projects they are members of — all others return `403`

### Input Validation & Sanitization

- All input validated through DRF serializers before reaching the database
- `bleach` sanitizes HTML in text fields to prevent XSS — allowed tags: `p`, `br`, `strong`, `em`, `ul`, `ol`, `li`, `h1`–`h6`
- `validators.py` blocks `<script>`, `javascript:`, and `on*=` patterns in project names
- Filename sanitization strips `..`, `/`, `\`, and null bytes to prevent directory traversal
- Dangerous extensions (`.exe`, `.bat`, `.cmd`, `.vbs`, `.jar`) explicitly blocked on upload

### Rate Limiting & CORS

| Setting | Value |
|---------|-------|
| Anonymous rate limit | 100 requests / hour |
| Authenticated rate limit | 1000 requests / hour |
| CORS allowed origins | `localhost:5173` only in development |
| `CORS_ALLOW_ALL_ORIGINS` | Only `True` when `DEBUG=True` |

### Known Limitations

| Limitation | Production Fix |
|------------|---------------|
| Both tokens in `localStorage` | `HttpOnly` cookie for refresh token |
| Token blacklisting disabled | Enable `BLACKLIST_AFTER_ROTATION` |
| No HTTPS locally | TLS via nginx reverse proxy |
| SQLite — no row-level encryption | PostgreSQL with encrypted storage |
| No brute-force protection on login | Per-IP throttle on `/auth/login/` |

---

## 10. 🤖 AI Usage

AI tools (Amazon Q Developer / GitHub Copilot) were used during development.

**Where AI helped:**

| Area | What AI Did |
|------|-------------|
| Boilerplate | Generated repetitive serializers, URL patterns, and TanStack Query hooks |
| Debugging | Fixed CORS errors between Vite and Django; resolved JWT refresh race condition on concurrent 401s |
| Documentation | Generated initial drafts, reviewed and corrected against actual code |

**Where AI was NOT used:**
- Data model design — hierarchy, field choices, and FK relationships designed manually
- Background job retry logic — exponential backoff in `run_job_with_tracking()` written manually
- Security decisions — CORS config, JWT lifetimes, permission classes, rate limiting
- Architecture decisions — APScheduler vs Celery, polling vs WebSockets, TanStack Query + Zustand split

> Every AI output was read, tested against the running application, and corrected where inaccurate. AI accelerated repetitive work — all core design and engineering judgment is my own.

---

## 11. 🔮 What I'd Improve or Build Next

### 🔴 High Priority
| Improvement | Why |
|-------------|-----|
| PostgreSQL | Concurrent writes, row-level locking, FTS, encrypted storage |
| Celery + Redis | Distributed task queue, proper retries, Flower monitoring |
| WebSockets (Django Channels) | Real-time board updates, live notifications, presence indicators |
| `HttpOnly` cookie for refresh token | Eliminate XSS risk of `localStorage` |

### 🟡 Medium Priority
| Improvement | Why |
|-------------|-----|
| Sprint planning view | Backlog management, drag stories into sprints, velocity tracking |
| Burndown charts | Visual sprint progress — `story_points` field already exists on the model |
| Email notifications | Digest emails for deadlines and invitations via background job |
| Full-text search | PostgreSQL FTS or Elasticsearch across descriptions and comments |
| PDF report export | ReportLab is already installed — just needs the template |

### 🟢 Nice to Have
| Improvement | Why |
|-------------|-----|
| GitHub / GitLab integration | Link commits to tasks, auto-transition status on PR merge |
| Mobile app (React Native) | Same REST API, native mobile experience |
| Two-factor authentication | TOTP-based 2FA for admin accounts |
| Offline support | Service worker + IndexedDB cache with sync on reconnect |
| Audit log | Full mutation history — `ActivityLog` model is already partially in place |
| Docker + Docker Compose | One-command setup for the full stack |

---

Built as a Full-Stack Intern Assignment · Django + React · SQLite · APScheduler

