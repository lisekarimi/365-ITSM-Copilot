# Enterprise RBAC

This demo uses **maker-provided** credentials — all users share the maker's ServiceNow access. For a proper enterprise role-based access control setup:

1. **Entra ID** — create security groups (e.g. `ITSM-Users`, `ITSM-Admins`).
2. **Copilot Studio** — restrict agent access to members of `ITSM-Users` under **Settings → Security → Authentication**.
3. **ServiceNow** — switch from maker credentials to **end-user authentication**, so each user only sees their own data.
