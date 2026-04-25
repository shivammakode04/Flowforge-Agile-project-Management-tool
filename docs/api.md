# API Documentation

**Base URL:** `http://localhost:8000/api`  
**Auth:** All endpoints require `Authorization: Bearer <access_token>` unless marked public.  
**Interactive Docs:** `http://localhost:8000/api/docs/` (Swagger) · `http://localhost:8000/api/redoc/` (ReDoc)

---

## Authentication

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/auth/register/` | Public | Register new user |
| POST | `/auth/login/` | Public | Login — returns `access` + `refresh` tokens |
| POST | `/auth/refresh/` | Public | Refresh access token |
| GET | `/auth/profile/` | Required | Get current user profile |
| PATCH | `/auth/profile/` | Required | Update profile (email, full_name) |
| POST | `/auth/profile/avatar/` | Required | Upload avatar image (multipart, max 5MB) |
| POST | `/auth/profile/password/` | Required | Change password |

---

## Users

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/auth/users/` | List all users. Filters: `?search=`, `?role=`, `?is_active=` |
| GET | `/auth/users/:id/` | Get user detail with task/project stats |
| PATCH | `/auth/users/:id/role/` | Change user's global role — admin only |
| POST | `/auth/users/:id/toggle-active/` | Activate / deactivate user — admin only |
| GET | `/auth/users/:id/workload/` | Task workload summary for a user |
| GET | `/auth/my/tasks/` | Current user's assigned tasks. Filters: `?status=`, `?priority=`, `?project=` |
| GET | `/auth/my/stats/` | Dashboard stats (project count, task counts, overdue) |
| GET | `/auth/my/activity/` | Activity feed across all user's projects |

---

## Projects

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/projects/` | List user's projects |
| POST | `/projects/` | Create project — creator auto-added as admin |
| GET | `/projects/:id/` | Get project detail |
| PATCH | `/projects/:id/` | Update project — project admin only |
| DELETE | `/projects/:id/` | Delete project — project admin only |
| POST | `/projects/:id/archive/` | Toggle archive status — project admin only |
| GET | `/projects/:id/members/` | List members. Filter: `?status=accepted\|pending` |
| POST | `/projects/:id/members/` | Add member directly (status: accepted) — project admin only |
| DELETE | `/projects/:id/members/:user_id/` | Remove member — project admin only |
| PATCH | `/projects/:id/members/:user_id/role/` | Change member role — project admin only |
| POST | `/projects/:id/invite/` | Invite user (status: pending) — project admin only |
| POST | `/projects/invites/:member_id/accept/` | Accept a project invitation |
| GET | `/projects/:id/activity/` | Project activity log |
| GET | `/projects/:id/analytics/` | Task/story stats, completion rate, hours |
| GET | `/projects/:id/stories/` | All stories in project |
| GET | `/projects/:id/tasks/` | All tasks in project (Kanban board data) |
| GET | `/projects/:id/export/` | Export project — `?format=csv` (default) or `?format=json` |
| POST | `/projects/:id/report/` | Trigger async background report — returns `202 Accepted` |

---

## User Stories

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/projects/:project_id/stories/` | List stories in a project |
| POST | `/projects/:project_id/stories/` | Create story — member or admin only |
| GET | `/stories/:id/` | Get story detail |
| PATCH | `/stories/:id/` | Update story — admin (any), member (own only) |
| DELETE | `/stories/:id/` | Delete story — admin (any), member (own only) |

**Story fields:** `title`, `description`, `acceptance_criteria`, `status`, `priority`, `story_points`, `business_value`  
**Status values:** `todo` · `in_progress` · `done` · `blocked` · `testing`  
**Priority values:** `low` · `medium` · `high` · `critical`

---

## Tasks

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/stories/:story_id/tasks/` | List tasks in a story. Filters: `?status=`, `?priority=`, `?assigned_to=` |
| POST | `/stories/:story_id/tasks/` | Create task — member or admin only |
| GET | `/tasks/:id/` | Get task detail |
| PATCH | `/tasks/:id/` | Update task |
| DELETE | `/tasks/:id/` | Delete task |
| PATCH | `/tasks/:id/status/` | Update task status |
| POST | `/tasks/:id/assign/` | Replace assignee list — body: `{ "assigned_to": [1, 2] }` |
| GET | `/tasks/my/` | Tasks assigned to current user |
| GET | `/projects/:project_id/tasks/` | All tasks in a project |

**Task fields:** `title`, `description`, `status`, `priority`, `task_type`, `assigned_to`, `due_date`, `estimated_hours`, `actual_hours`, `labels`, `is_blocked`  
**Task type values:** `feature` · `bug` · `enhancement` · `research` · `testing`

---

## Comments & Attachments

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/tasks/:task_id/comments/` | List comments on a task |
| POST | `/tasks/:task_id/comments/` | Add comment — body: `{ "content": "..." }` |
| DELETE | `/comments/:id/` | Delete own comment |
| GET | `/tasks/:task_id/attachments/` | List attachments |
| POST | `/tasks/:task_id/attachments/` | Upload file (multipart) — member or admin only, max 5MB |

---

## Notifications

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/notifications/` | List all notifications for current user |
| PATCH | `/notifications/:id/` | Mark notification as read |
| POST | `/notifications/mark-all-read/` | Mark all as read |
| GET | `/notifications/unread-count/` | Get unread count — used for badge polling |

---

## Teams

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/teams/` | List teams (owned or member of) |
| POST | `/teams/` | Create team |
| GET/PATCH/DELETE | `/teams/:id/` | Get, update, or delete team |
| POST | `/teams/:id/invite/` | Invite user to team — body: `{ "user_id": 3 }` |
| GET | `/invitations/` | List pending team invitations for current user |
| POST | `/invitations/:id/accept/` | Accept team invitation |
| POST | `/invitations/:id/reject/` | Reject team invitation |

---

## Search

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/search/` | Search across projects, stories, and tasks. Params: `?q=`, `?project_id=`, `?status=`, `?priority=`, `?assigned_to=` |

Returns up to 10 projects, 10 stories, and 20 tasks scoped to the user's memberships.

---

## Background Jobs

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/jobs/` | List all background job records |
| POST | `/jobs/trigger/` | Manually trigger a job — body: `{ "job_type": "deadline_reminder" }` |

**Supported job types:** `deadline_reminder` · `notification_cleanup`  
**Job status values:** `pending` · `running` · `completed` · `failed`

---

## Common Status Codes

| Code | Meaning |
|------|---------|
| `200` | OK |
| `201` | Created |
| `202` | Accepted (async job started) |
| `204` | No Content (delete success) |
| `400` | Bad Request (validation error) |
| `401` | Unauthorized (missing or invalid token) |
| `403` | Forbidden (insufficient role) |
| `404` | Not Found |
| `500` | Internal Server Error |
