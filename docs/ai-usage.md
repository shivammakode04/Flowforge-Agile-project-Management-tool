# AI Usage

AI tools (Amazon Q Developer / GitHub Copilot) were used during development. This is an honest account of where and how.

---

## Where AI Helped

| Area | What AI Did |
|------|-------------|
| Boilerplate | Generated repetitive serializers, URL patterns, and TanStack Query hooks that follow the same pattern across apps |
| Debugging | Diagnosed CORS errors between Vite and Django; fixed a race condition in the Axios JWT refresh interceptor on concurrent 401s |
| Documentation | Generated initial drafts of docs, which were then reviewed and corrected against the actual code |

## Where AI Was Not Used

- Data model design — hierarchy, field choices, and relationships were designed manually
- Background job retry logic — exponential backoff in `run_job_with_tracking()` was written manually
- Security decisions — CORS config, JWT lifetimes, permission classes, and rate limiting were deliberate manual choices
- Architecture decisions — APScheduler vs Celery, polling vs WebSockets, TanStack Query + Zustand split

## Validation

Every AI output was read, tested against the running application, and corrected where inaccurate or inconsistent with the codebase. AI accelerated repetitive work — all core design and engineering judgment is my own.
