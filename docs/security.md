# Security Considerations

---

## Authentication

- **JWT via SimpleJWT** — access tokens expire in **1 hour**, refresh tokens expire in **7 days** with rotation enabled (`ROTATE_REFRESH_TOKENS = True`). Each refresh issues a new refresh token.
- **Token blacklisting** is currently disabled (`BLACKLIST_AFTER_ROTATION = False`). This means rotated refresh tokens are not invalidated server-side. Enabling the simplejwt blacklist app would fix this.
- Both access and refresh tokens are stored together in `localStorage` under the key `flowforge_tokens` as a JSON object (`{ access, refresh }`). This means **both tokens are XSS-accessible**. The Axios interceptor reads from `localStorage` on every request and auto-refreshes on 401.

---

## Authorization & Permissions

Three custom DRF permission classes enforce object-level access:

| Class | Grants access when |
|---|---|
| `IsProjectMember` | User has any `ProjectMember` record for the project |
| `IsProjectMemberRole` | User has `role=admin` or `role=member` in the project |
| `IsProjectAdmin` | User has `role=admin` in the project |

- Viewers can read but cannot create, update, or delete any resource.
- Members can create/edit stories and tasks they own; cannot manage project membership.
- Project admins can manage members, roles, archive, and delete the project.
- Global admins (user `role=admin`) can manage all users and change global roles. Self-demotion and self-deactivation are explicitly blocked in the view logic.
- Users can only access projects they are members of — attempting to access another user's project returns `403`.

---

## Input Validation

- **DRF serializers** validate all incoming data before it reaches the database — type checking, required fields, max lengths, and choice validation.
- **`bleach`** sanitizes HTML in text fields (project descriptions, comments) to strip disallowed tags and prevent XSS. Allowed tags: `p`, `br`, `strong`, `em`, `ul`, `ol`, `li`, `h1`–`h6`.
- **`validators.py`** enforces additional rules:
  - Project names: min 3, max 100 characters; blocks `<script>`, `javascript:`, and `on*=` patterns.
  - Story points: 0–100 integer only.
  - Hours: 0–999 float only.
- **Filename sanitization** in `security.py` strips `..`, `/`, `\`, and null bytes to prevent directory traversal on file uploads.
- Dangerous executable extensions (`.exe`, `.bat`, `.cmd`, `.vbs`, `.jar`, etc.) are explicitly blocked on upload.

---

## File Uploads

- Max file size: **5 MB** (`DATA_UPLOAD_MAX_MEMORY_SIZE` and `FILE_UPLOAD_MAX_MEMORY_SIZE` in settings).
- Allowed MIME types for attachments: `image/jpeg`, `image/png`, `image/gif`, `image/webp`, `application/pdf`, `text/plain`, `application/msword`, `application/vnd.openxmlformats-officedocument.wordprocessingml.document`.
- Allowed MIME types for avatars: `image/jpeg`, `image/png`, `image/gif`, `image/webp` only.
- Both MIME type and file extension are validated independently.

---

## Rate Limiting

Configured via DRF's built-in throttle classes in `settings.py`:

| Client type | Limit |
|---|---|
| Anonymous | 100 requests / hour |
| Authenticated | 1000 requests / hour |

---

## CORS

- Allowed origins restricted to `http://localhost:5173` and `http://127.0.0.1:5173` in development via `django-cors-headers`.
- `CORS_ALLOW_ALL_ORIGINS` is tied to `DEBUG` — it is only `True` when `DEBUG=True`. Must be explicitly set to `False` in production with a strict `CORS_ALLOWED_ORIGINS` list.

---

## Secrets Management

- `SECRET_KEY`, `DEBUG`, `ALLOWED_HOSTS`, and `CORS_ALLOWED_ORIGINS` are loaded from a `.env` file via `python-dotenv`. Nothing sensitive is hardcoded.
- `.env` is excluded from version control via `.gitignore`.

---

## Known Limitations

| Limitation | Production Fix |
|---|---|
| Refresh token in `localStorage` | Use `HttpOnly` cookie |
| Token blacklisting disabled | Enable `BLACKLIST_AFTER_ROTATION` with simplejwt blacklist app |
| No HTTPS enforced locally | TLS termination via nginx reverse proxy in production |
| SQLite has no row-level encryption | PostgreSQL with encrypted storage at rest |
| No login attempt rate limiting | Add per-IP throttle on `/auth/login/` |
