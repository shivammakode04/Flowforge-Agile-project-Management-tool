# Database Schema

**Engine:** SQLite · **ORM:** Django 4.2

---

## Entity Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                                                                 │
│   ┌──────────────┐        ┌───────────────────┐        ┌──────────────────┐    │
│   │    users     │        │  project_members  │        │    projects      │    │
│   ├──────────────┤        ├───────────────────┤        ├──────────────────┤    │
│   │ PK id        │◄───────│ FK user_id        │───────►│ PK id            │    │
│   │    username  │        │ FK project_id     │        │    name          │    │
│   │    email     │        │    role           │        │ FK owner_id      │───►│
│   │    full_name │        │    status         │        │    status        │    │
│   │    avatar_url│        │    joined_at      │        │    is_archived   │    │
│   │    role      │        └───────────────────┘        │    start_date    │    │
│   │    is_active │                                      │    end_date      │    │
│   │    created_at│◄─────────────────────────────────── │    est_hours     │    │
│   └──────┬───────┘  owner                              │    actual_hours  │    │
│          │                                             │    created_at    │    │
│          │                                             └────────┬─────────┘    │
│          │                                                      │              │
│          │                                             ┌────────▼─────────┐    │
│          │                                             │   user_stories   │    │
│          │                                             ├──────────────────┤    │
│          │                                             │ PK id            │    │
│          │                                             │ FK project_id    │    │
│          │                                             │ FK created_by_id │───►│
│          │                                             │    title         │    │
│          │                                             │    description   │    │
│          │                                             │    acceptance_   │    │
│          │                                             │    criteria      │    │
│          │                                             │    status        │    │
│          │                                             │    priority      │    │
│          │                                             │    story_points  │    │
│          │                                             │    business_value│    │
│          │                                             │    created_at    │    │
│          │                                             └────────┬─────────┘    │
│          │                                                      │              │
│          │                                             ┌────────▼─────────┐    │
│          │                ┌──────────────────┐         │      tasks       │    │
│          │                │  task_assigned_to│         ├──────────────────┤    │
│          │◄───────────────│ FK user_id       │─────────│ PK id            │    │
│          │                │ FK task_id       │         │ FK story_id      │    │
│          │                └──────────────────┘         │ FK created_by_id │───►│
│          │                                             │    title         │    │
│          │                ┌──────────────────┐         │    description   │    │
│          │◄───────────────│ task_completed_by│─────────│    status        │    │
│          │                │ FK user_id       │         │    priority      │    │
│          │                │ FK task_id       │         │    task_type     │    │
│          │                └──────────────────┘         │    due_date      │    │
│          │                                             │    est_hours     │    │
│          │         ┌──────────────┐                    │    actual_hours  │    │
│          │         │   comments   │                    │    labels (JSON) │    │
│          │         ├──────────────┤                    │    is_blocked    │    │
│          │◄────────│ FK user_id   │◄───────────────────│    created_at    │    │
│          │         │ FK task_id   │                    └────────┬─────────┘    │
│          │         │    content   │                             │              │
│          │         │    created_at│             ┌───────────────┘              │
│          │         └──────────────┘             │                              │
│          │                                      │                              │
│          │         ┌──────────────────┐         │                              │
│          │         │ task_attachments │         │                              │
│          │         ├──────────────────┤         │                              │
│          │◄────────│ FK uploaded_by_id│◄────────┘                              │
│          │         │ FK task_id       │                                        │
│          │         │    file          │                                        │
│          │         │    filename      │                                        │
│          │         │    file_size     │                                        │
│          │         │    content_type  │                                        │
│          │         │    created_at    │                                        │
│          │         └──────────────────┘                                        │
│          │                                                                     │
│          │         ┌──────────────────┐    ┌──────────────────────────┐        │
│          │         │  notifications   │    │      activity_logs       │        │
│          │         ├──────────────────┤    ├──────────────────────────┤        │
│          │◄────────│ FK user_id       │    │ FK user_id          ◄────┼────────│
│          │         │ FK project_id    │    │ FK project_id            │        │
│          │         │    type          │    │    action                │        │
│          │         │    title         │    │    target_type           │        │
│          │         │    message       │    │    target_id             │        │
│          │         │    is_read       │    │    created_at            │        │
│          │         │    related_obj_id│    └──────────────────────────┘        │
│          │         │    created_at    │                                        │
│          │         └──────────────────┘                                        │
│          │                                                                     │
│          │         ┌──────────────┐    ┌──────────────────────┐                │
│          │         │    teams     │    │   team_invitations   │                │
│          │         ├──────────────┤    ├──────────────────────┤                │
│          │◄────────│ FK owner_id  │    │ FK team_id      ◄────┼────────────────│
│          │◄────────│ members (M2M)│    │ FK inviter_id   ◄────┼────────────────│
│          │         │    name      │    │ FK invitee_id   ◄────┼────────────────│
│          │         │    created_at│    │    status            │                │
│          │         └──────────────┘    │    created_at        │                │
│          │                            └──────────────────────┘                │
│          │                                                                     │
│          │         ┌──────────────────┐                                        │
│          │         │  background_jobs │  (standalone — no FK)                  │
│          │         ├──────────────────┤                                        │
│          │         │ PK id            │                                        │
│          │         │    job_type      │                                        │
│          │         │    status        │                                        │
│          │         │    payload       │                                        │
│          │         │    result        │                                        │
│          │         │    retry_count   │                                        │
│          │         │    max_retries   │                                        │
│          │         │    error_message │                                        │
│          │         │    scheduled_at  │                                        │
│          │         │    executed_at   │                                        │
│          │         └──────────────────┘                                        │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Relationship Summary

| Relationship | Type | Details |
|---|---|---|
| `users` → `projects` | One-to-Many | A user owns many projects (`owner_id`) |
| `users` ↔ `projects` | Many-to-Many | Via `project_members` junction table |
| `projects` → `user_stories` | One-to-Many | A project has many stories (`project_id`) |
| `user_stories` → `tasks` | One-to-Many | A story has many tasks (`story_id`) |
| `users` ↔ `tasks` | Many-to-Many | Via `task_assigned_to` (assignees) |
| `users` ↔ `tasks` | Many-to-Many | Via `task_completed_by` (completion tracking) |
| `tasks` → `comments` | One-to-Many | A task has many comments (`task_id`) |
| `tasks` → `task_attachments` | One-to-Many | A task has many attachments (`task_id`) |
| `projects` → `activity_logs` | One-to-Many | A project has many activity entries |
| `users` → `notifications` | One-to-Many | A user has many notifications |
| `users` → `teams` | One-to-Many | A user owns many teams (`owner_id`) |
| `users` ↔ `teams` | Many-to-Many | Via `team_members` junction table |
| `teams` → `team_invitations` | One-to-Many | A team has many invitations |

---

## Table Definitions

### `users`

| Column | Type | Constraints | Default |
|--------|------|-------------|---------|
| `id` | INTEGER | PK, auto | — |
| `username` | VARCHAR(150) | UNIQUE, NOT NULL | — |
| `email` | VARCHAR(254) | UNIQUE, NOT NULL | — |
| `full_name` | VARCHAR(255) | NOT NULL | `''` |
| `avatar_url` | VARCHAR | NULL | — |
| `role` | VARCHAR(20) | NOT NULL | `member` |
| `is_active` | BOOLEAN | NOT NULL | `True` |
| `password` | VARCHAR | NOT NULL | — |
| `created_at` | DATETIME | NOT NULL, auto_now_add | — |

`role` choices: `admin` · `member` · `viewer`

---

### `projects`

| Column | Type | Constraints | Default |
|--------|------|-------------|---------|
| `id` | INTEGER | PK, auto | — |
| `name` | VARCHAR(255) | NOT NULL | — |
| `description` | TEXT | NOT NULL | `''` |
| `owner_id` | INTEGER | FK → `users.id` CASCADE | — |
| `is_archived` | BOOLEAN | NOT NULL | `False` |
| `status` | VARCHAR(20) | NOT NULL | `planning` |
| `start_date` | DATE | NULL | — |
| `end_date` | DATE | NULL | — |
| `estimated_hours` | INTEGER | NULL | — |
| `actual_hours` | INTEGER | NULL | — |
| `created_at` | DATETIME | NOT NULL, auto_now_add | — |
| `updated_at` | DATETIME | NOT NULL, auto_now | — |

`status` choices: `planning` · `active` · `on_hold` · `completed` · `cancelled`

---

### `project_members`

| Column | Type | Constraints | Default |
|--------|------|-------------|---------|
| `id` | INTEGER | PK, auto | — |
| `project_id` | INTEGER | FK → `projects.id` CASCADE | — |
| `user_id` | INTEGER | FK → `users.id` CASCADE | — |
| `role` | VARCHAR(20) | NOT NULL | `viewer` |
| `status` | VARCHAR(20) | NOT NULL | `pending` |
| `joined_at` | DATETIME | NOT NULL, auto_now_add | — |

`role` choices: `admin` · `member` · `viewer`  
`status` choices: `pending` · `accepted`  
Unique constraint: `(project_id, user_id)`

---

### `user_stories`

| Column | Type | Constraints | Default |
|--------|------|-------------|---------|
| `id` | INTEGER | PK, auto | — |
| `project_id` | INTEGER | FK → `projects.id` CASCADE | — |
| `title` | VARCHAR(255) | NOT NULL | — |
| `description` | TEXT | NOT NULL | `''` |
| `acceptance_criteria` | TEXT | NOT NULL | `''` |
| `status` | VARCHAR(20) | NOT NULL | `todo` |
| `priority` | VARCHAR(20) | NOT NULL | `medium` |
| `story_points` | INTEGER | NULL | — |
| `business_value` | VARCHAR(50) | NOT NULL | `medium` |
| `created_by_id` | INTEGER | FK → `users.id` SET NULL, NULL | — |
| `created_at` | DATETIME | NOT NULL, auto_now_add | — |
| `updated_at` | DATETIME | NOT NULL, auto_now | — |

`status` choices: `todo` · `in_progress` · `done` · `blocked` · `testing`  
`priority` choices: `low` · `medium` · `high` · `critical`  
`business_value` choices: `low` · `medium` · `high`  
Default ordering: `-priority`, `created_at`

---

### `tasks`

| Column | Type | Constraints | Default |
|--------|------|-------------|---------|
| `id` | INTEGER | PK, auto | — |
| `story_id` | INTEGER | FK → `user_stories.id` CASCADE | — |
| `title` | VARCHAR(255) | NOT NULL | — |
| `description` | TEXT | NOT NULL | `''` |
| `status` | VARCHAR(20) | NOT NULL | `todo` |
| `priority` | VARCHAR(20) | NOT NULL | `medium` |
| `task_type` | VARCHAR(50) | NOT NULL | `feature` |
| `due_date` | DATE | NULL | — |
| `estimated_hours` | DECIMAL(5,2) | NULL | — |
| `actual_hours` | DECIMAL(5,2) | NULL | — |
| `labels` | JSON | NOT NULL | `[]` |
| `is_blocked` | BOOLEAN | NOT NULL | `False` |
| `created_by_id` | INTEGER | FK → `users.id` SET NULL, NULL | — |
| `created_at` | DATETIME | NOT NULL, auto_now_add | — |
| `updated_at` | DATETIME | NOT NULL, auto_now | — |

`status` choices: `todo` · `in_progress` · `done` · `blocked` · `testing`  
`priority` choices: `low` · `medium` · `high` · `critical`  
`task_type` choices: `feature` · `bug` · `enhancement` · `research` · `testing`  
M2M: `assigned_to` → `users` · `completed_by` → `users`

---

### `comments`

| Column | Type | Constraints | Default |
|--------|------|-------------|---------|
| `id` | INTEGER | PK, auto | — |
| `task_id` | INTEGER | FK → `tasks.id` CASCADE | — |
| `user_id` | INTEGER | FK → `users.id` CASCADE | — |
| `content` | TEXT | NOT NULL | — |
| `created_at` | DATETIME | NOT NULL, auto_now_add | — |

Default ordering: `created_at` (ascending)

---

### `task_attachments`

| Column | Type | Constraints | Default |
|--------|------|-------------|---------|
| `id` | INTEGER | PK, auto | — |
| `task_id` | INTEGER | FK → `tasks.id` CASCADE | — |
| `file` | VARCHAR | NOT NULL | `media/task_attachments/` |
| `filename` | VARCHAR(255) | NOT NULL | — |
| `file_size` | INTEGER (unsigned) | NOT NULL | — |
| `content_type` | VARCHAR(100) | NOT NULL | `application/octet-stream` |
| `uploaded_by_id` | INTEGER | FK → `users.id` CASCADE | — |
| `created_at` | DATETIME | NOT NULL, auto_now_add | — |

Allowed types: JPEG · PNG · GIF · WebP · PDF · TXT · DOC · DOCX · Max size: 5 MB

---

### `notifications`

| Column | Type | Constraints | Default |
|--------|------|-------------|---------|
| `id` | INTEGER | PK, auto | — |
| `user_id` | INTEGER | FK → `users.id` CASCADE | — |
| `type` | VARCHAR(50) | NOT NULL | — |
| `title` | VARCHAR(255) | NOT NULL | `Notification` |
| `message` | TEXT | NOT NULL | — |
| `is_read` | BOOLEAN | NOT NULL | `False` |
| `project_id` | INTEGER | FK → `projects.id` CASCADE, NULL | — |
| `related_object_id` | INTEGER | NULL | — |
| `created_at` | DATETIME | NOT NULL, auto_now_add | — |

`type` choices: `task_assigned` · `task_updated` · `task_completed` · `story_assigned` · `story_updated` · `project_invite` · `team_invitation` · `team_invitation_accepted` · `deadline_reminder` · `comment_added` · `project_updated`

---

### `activity_logs`

| Column | Type | Constraints | Default |
|--------|------|-------------|---------|
| `id` | INTEGER | PK, auto | — |
| `user_id` | INTEGER | FK → `users.id` SET NULL, NULL | — |
| `project_id` | INTEGER | FK → `projects.id` CASCADE | — |
| `action` | VARCHAR(255) | NOT NULL | — |
| `target_type` | VARCHAR(50) | NULL | — |
| `target_id` | INTEGER | NULL | — |
| `created_at` | DATETIME | NOT NULL, auto_now_add | — |

`target_type` examples: `task` · `story` · `project` · `member` · `comment` · `attachment`

---

### `background_jobs`

| Column | Type | Constraints | Default |
|--------|------|-------------|---------|
| `id` | INTEGER | PK, auto | — |
| `job_type` | VARCHAR(100) | NOT NULL | — |
| `status` | VARCHAR(20) | NOT NULL | `pending` |
| `payload` | TEXT | NULL | — |
| `result` | TEXT | NULL | — |
| `retry_count` | INTEGER | NOT NULL | `0` |
| `max_retries` | INTEGER | NOT NULL | `3` |
| `error_message` | TEXT | NULL | — |
| `scheduled_at` | DATETIME | NOT NULL, auto_now_add | — |
| `executed_at` | DATETIME | NULL | — |
| `created_at` | DATETIME | NOT NULL, auto_now_add | — |

`status` choices: `pending` · `running` · `completed` · `failed`  
No foreign keys — standalone audit table.

---

### `teams`

| Column | Type | Constraints | Default |
|--------|------|-------------|---------|
| `id` | INTEGER | PK, auto | — |
| `name` | VARCHAR(255) | NOT NULL | — |
| `owner_id` | INTEGER | FK → `users.id` CASCADE | — |
| `created_at` | DATETIME | NOT NULL, auto_now_add | — |

M2M: `members` → `users`

---

### `team_invitations`

| Column | Type | Constraints | Default |
|--------|------|-------------|---------|
| `id` | INTEGER | PK, auto | — |
| `team_id` | INTEGER | FK → `teams.id` CASCADE | — |
| `inviter_id` | INTEGER | FK → `users.id` CASCADE | — |
| `invitee_id` | INTEGER | FK → `users.id` CASCADE | — |
| `status` | VARCHAR(20) | NOT NULL | `pending` |
| `created_at` | DATETIME | NOT NULL, auto_now_add | — |
| `updated_at` | DATETIME | NOT NULL, auto_now | — |

`status` choices: `pending` · `accepted` · `rejected`  
Unique constraint: `(team_id, invitee_id)`
