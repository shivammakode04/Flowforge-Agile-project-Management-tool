# Design Decisions & Tradeoffs

---

## 1. SQLite over PostgreSQL

**Decision:** SQLite as the primary database.

**Reason:** The assignment targets small teams (3–10 users). SQLite requires no separate server process, zero configuration, and is fully supported by Django's ORM. Switching to PostgreSQL later is a single line change in `settings.py`.

**Tradeoff:** SQLite does not handle concurrent writes well. Under high write load or multiple simultaneous users mutating data, it will become a bottleneck. Not suitable for production at scale.

---

## 2. APScheduler over Celery

**Decision:** APScheduler running in-process for background jobs.

**Reason:** Celery requires a Redis or RabbitMQ broker — additional infrastructure that adds setup complexity beyond the assignment scope. APScheduler runs inside the Django process and satisfies the async requirement with no external dependencies.

**Tradeoff:** In-process jobs share the web server's memory and CPU. Jobs do not survive server restarts. There is no distributed worker pool. For production, Celery + Redis would be the correct choice.

---

## 3. Polling over WebSockets

**Decision:** Notifications polled every 30 seconds via `setInterval`. Kanban board refetched every 60 seconds via TanStack Query `refetchInterval`.

**Reason:** WebSockets require Django Channels and a Redis channel layer — more infrastructure. For a small team tool, a 30-second notification delay is acceptable and polling is far simpler to implement and debug.

**Tradeoff:** Not real-time. A user could see up to a 30-second delay on notifications. Collaborative editing conflicts are not handled.

---

## 4. Both Tokens in localStorage

**Decision:** Both the access token and refresh token are stored together in `localStorage` under the key `flowforge_tokens` as a JSON object. Axios interceptors auto-refresh on 401 and retry the original request.

**Reason:** Stateless auth fits the decoupled SPA architecture. The interceptor handles token rotation transparently without the user noticing. `localStorage` persists across tab closes, keeping the user logged in.

**Tradeoff:** Both tokens are in `localStorage`, which is accessible to JavaScript and therefore vulnerable to XSS. The production fix is storing the refresh token in an `HttpOnly` cookie and the access token in memory only.

---

## 5. TanStack Query + Zustand Split

**Decision:** TanStack Query manages all server state (projects, stories, tasks). Zustand manages only global UI state (auth, theme, notification count).

**Reason:** TanStack Query provides caching, background refetch, loading/error states, and optimistic updates out of the box. Putting server state in Zustand would require manually replicating all of that. The split keeps each library doing what it does best.

**Tradeoff:** Two state libraries adds a small learning curve. Developers need to know which store owns which piece of state.

---

## 6. Optimistic Updates on Drag-and-Drop

**Decision:** Kanban drag-and-drop updates the TanStack Query cache immediately before the API call completes.

**Reason:** Gives instant visual feedback. If the API call fails, the cache is rolled back and a toast error is shown to the user.

**Tradeoff:** Brief inconsistency between UI and server state on failure. Acceptable given the rollback mechanism is in place.

---

## 7. Flat Three-Level Hierarchy

**Decision:** Strict Project → UserStory → Task. No sub-tasks, no cross-project stories.

**Reason:** Matches the assignment requirement exactly. Reflects standard agile workflows (epics/stories/tasks). Keeping it flat makes queries, permissions, and the UI significantly simpler.

**Tradeoff:** No support for nested tasks or linking a story to multiple projects. A more flexible graph model would add complexity without benefit at this scope.

---

## 8. Per-Project Role + Global Role

**Decision:** Two-level role system — a global user role (`admin`/`member`/`viewer`) and a separate per-project membership role (`admin`/`member`/`viewer`).

**Reason:** A global admin needs to manage all users across the workspace. A project admin only needs control over their own project's members. Separating the two avoids giving project admins workspace-wide power.

**Tradeoff:** Two role fields to check in permission logic. No fine-grained per-story or per-task permissions, which is acceptable for a small team tool.
