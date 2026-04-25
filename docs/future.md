# What I'd Improve or Build Next

---

## Infrastructure & Backend

| Idea | Why |
|------|-----|
| Replace SQLite with PostgreSQL | Concurrent writes, row-level locking, full-text search, encrypted storage at rest |
| Replace APScheduler with Celery + Redis | Distributed task queue, proper retry management, worker scaling, Flower monitoring UI |
| Add token blacklisting | Enable `BLACKLIST_AFTER_ROTATION` in SimpleJWT to invalidate rotated refresh tokens server-side |
| Move refresh token to `HttpOnly` cookie | Eliminates XSS risk of storing refresh token in `localStorage` |
| Add HTTPS + nginx reverse proxy | TLS termination, static file serving, rate limiting at the proxy layer |
| Per-endpoint rate limiting on `/auth/login/` | Prevent brute-force attacks on the login endpoint specifically |
| Database connection pooling (pgBouncer) | Efficient connection reuse under concurrent load with PostgreSQL |
| Containerize with Docker + Docker Compose | One-command setup for backend, frontend, DB, Redis, and worker |

---

## Real-Time & Collaboration

| Idea | Why |
|------|-----|
| WebSockets via Django Channels | Replace 30-second notification polling with instant push — task updates, comments, status changes appear live |
| Presence indicators | Show which team members are currently viewing the same board or story |
| Collaborative editing on descriptions | Real-time co-editing of task and story descriptions, similar to Notion |
| Live Kanban board sync | Board updates pushed to all members instantly without manual refresh |

---

## Agile Features

| Idea | Why |
|------|-----|
| Sprint planning view | Backlog management, drag stories into sprints, set sprint dates and capacity |
| Burndown charts | Visual sprint progress using the existing `story_points` field already on the model |
| Velocity tracking | Track story points completed per sprint to forecast future capacity |
| Backlog prioritization | Drag-and-drop ordering of stories in the backlog with rank persistence |
| Story dependencies | Link stories that are blocked by other stories, visualize dependency graph |
| Sub-tasks | Allow tasks to have child tasks for finer-grained work breakdown |
| Time logging | Let users log time entries against tasks, not just set `actual_hours` manually |
| Custom workflows | Let project admins define their own status columns beyond the fixed five |

---

## Notifications & Communication

| Idea | Why |
|------|-----|
| Email notifications | Digest emails for deadline reminders, project invitations, and task assignments via background job |
| Slack / Teams integration | Post notifications to a team channel when tasks are assigned or completed |
| @mention in comments | Notify a specific user when mentioned in a task comment |
| Weekly project digest | Automated weekly summary email of project progress sent to all members |

---

## Reporting & Analytics

| Idea | Why |
|------|-----|
| PDF report export | Generate a formatted PDF project report using ReportLab (library already installed) |
| Burndown chart export | Export sprint burndown as an image or PDF |
| Custom date range analytics | Filter analytics by date range, not just all-time totals |
| Per-member contribution report | Tasks completed, hours logged, and stories contributed per team member |
| Project comparison dashboard | Side-by-side metrics across multiple projects |

---

## Search & Discovery

| Idea | Why |
|------|-----|
| Full-text search | PostgreSQL FTS or Elasticsearch across task descriptions, comments, and story content |
| Advanced filters | Filter tasks by label, assignee, date range, and type simultaneously |
| Saved filters | Let users save frequently used filter combinations |
| Global command palette | `Cmd+K` quick-jump to any project, story, or task by name |

---

## Integrations

| Idea | Why |
|------|-----|
| GitHub / GitLab integration | Link commits and PRs to tasks via webhook; auto-transition task status on PR merge |
| Jira import | Import existing projects and stories from Jira for teams migrating to FlowForge |
| Calendar sync | Export task due dates to Google Calendar or iCal |
| Zapier / webhook support | Let teams connect FlowForge events to any external tool |

---

## Frontend & UX

| Idea | Why |
|------|-----|
| Mobile-responsive layout | Current layout is desktop-first; a responsive design would improve usability on tablets |
| React Native mobile app | Native mobile app sharing the same REST API layer |
| Keyboard shortcuts | Full keyboard navigation for power users (already partially scaffolded in the frontend README) |
| Offline support | Service worker + IndexedDB cache for read-only offline access with sync on reconnect |
| Onboarding flow | Guided setup for new users — create first project, invite team, create first story |
| Dark mode improvements | Respect OS-level dark mode changes dynamically without page reload |

---

## Security & Compliance

| Idea | Why |
|------|-----|
| Full audit log | Track every data mutation (who changed what and when) — `ActivityLog` model is partially in place |
| Two-factor authentication | TOTP-based 2FA for admin accounts |
| SSO / OAuth2 | Login with Google or GitHub for teams that use those identity providers |
| Data export (GDPR) | Let users export all their personal data as JSON on request |
| Account deletion | Full data purge on user account deletion request |
