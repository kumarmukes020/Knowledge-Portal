# Online Knowledge Sharing Portal — Scaffold

## What's built (working end-to-end)

- **Database schema** (`database/schema.sql`) — every table from the spec: roles,
  employees, departments/locations/categories, knowledge + attachments,
  notifications, activity_logs, sessions, system_settings. Includes FKs,
  indexes, a FULLTEXT index for search, and seed data for roles/categories.
- **Auth + RBAC** (`includes/Auth.php`) — login by username or email, Argon2id
  password hashing with auto-rehash, account lockout after 5 failed attempts,
  session fixation protection (regenerate ID on login + periodically), idle
  timeout, `Auth::requireRole(...)` guard for any page.
- **CSRF protection** (`includes/Csrf.php`) — token per session, validated on
  every POST.
- **Audit logging** (`includes/ActivityLogger.php`) — writes to `activity_logs`
  for login/logout now; call `ActivityLogger::log()` the same way from every
  other module (knowledge create/update/delete, file upload, user management).
- **7-day edit/delete rule** (`includes/KnowledgeAccess.php`) — the exact
  `editable_until = submitted_date + 7 days` logic from the spec, centralized
  so the server-side check can never drift from the UI's button state.
- **Login / logout / role-aware dashboard** (`public/`) — real pages that
  exercise all of the above.
- **Security baseline** (`public/.htaccess`) — HTTPS redirect, no directory
  listing, PHP execution disabled inside the uploads folder (the #1 way file-
  upload apps get pwned), standard security headers.

## What's scaffolded but not yet built

Folders exist under `modules/` for: `employee`, `knowledge`, `dashboard` (full
charts), `reports`. Each should follow the same pattern already established:

1. `Auth::requireRole(...)` at the top of the page.
2. PDO prepared statements only — never string-concatenate SQL.
3. `Csrf::requireValid()` on every POST/PUT/DELETE handler.
4. `ActivityLogger::log(...)` after every create/update/delete.
5. For knowledge edit/delete pages specifically: call
   `KnowledgeAccess::canEditOrDelete()` server-side, not just to decide
   whether to show the button — enforce it before running the query.

## Setup

```bash
mysql -u root -p < database/schema.sql
```

Then in `database/schema.sql`, regenerate the seed Super User password hash:

```php
<?php
echo password_hash('YourRealPassword', PASSWORD_ARGON2ID);
```

Set environment variables (`DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASS`,
`APP_ENV`, `APP_URL`) rather than hardcoding credentials.

Point your web server's document root at `public/`.

## Suggested build order for the rest

1. **Knowledge submission form** (title, rich-text description via TinyMCE/
   CKEditor, category dropdown, multi-file upload with server-side MIME +
   extension + size validation, unique stored filenames).
2. **Public Knowledge Portal** (card/list view, FULLTEXT search + filters —
   the schema's `ft_know_search` index is ready for this).
3. **Employee Management** (Admin/Super User only — add/edit/deactivate).
4. **Dashboards with Chart.js** (pie/bar/line — the count queries in
   `dashboard.php` are the starting point; add GROUP BY category/department/
   month variants).
5. **Reports + Excel/PDF export**.
6. **Notifications** (cron or scheduled task scanning `editable_until` to
   populate the `notifications` table 24h before expiry).

Happy to build out any of these next — knowledge submission + the public
portal is probably the highest-value next step since it's the core loop.
