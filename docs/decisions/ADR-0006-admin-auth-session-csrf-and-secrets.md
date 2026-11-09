# ADR-0006: Admin Auth, Session Hardening, CSRF, and Secrets

- **Status:** Accepted (provisional CTO defaults, 2026-11-09)
- **Date:** 2026-11-09
- **Deciders:** CTO (provisional defaults approved for Phase 0 planning)

## Context
MVP has a single admin, CLI-provisioned, using Filament on `admin.mini-poll.com`. Admin auth/session/CSRF/secrets details were open.

## Decision
- **Admin provisioning:** create the single admin via **CLI only** (e.g., Laravel/Filament artisan command or dedicated seed/command). No public registration, no admin-creation API, no self-serve admin signup.
- **Authentication:** use **Filament/Laravel built-in auth** for the admin panel.
- **Password reset/recovery (MVP provisional):** no self-service email reset flow required (email notifications out of scope). Recovery/reset is an **operator action** via CLI/secure server access. Document the runbook in Phase 0.
- **Login throttling:** enable/rate-limit admin login attempts using Laravel/Fortify/Filament defaults or equivalent middleware.
- **Session hardening:** HTTPS-only cookies, `HttpOnly`, appropriate `SameSite` for admin web app, secure session configuration, APP_KEY/secret management, and session invalidation on logout.
- **CSRF:** enable Laravel CSRF protection for state-changing admin web/Filament requests. Public stateless `/api/v1/` endpoints are protected by validation, rate limiting, dedup/idempotency, and visibility rules rather than browser-session CSRF tokens (Phase 0 confirms exact mechanism).
- **Secrets/config:** all secrets (APP_KEY, DB credentials, any app-level tokens) in environment/config management, **never committed**; never exposed to browsers. Use Docker/runtime secret injection appropriate to deployment.

## Consequences
- Simple, secure-enough MVP posture for a single operator.
- Lack of self-service reset is acceptable only with disciplined operator access; revisit if admin model expands.
- Phase 0 must verify Filament route/domain configuration on `admin.mini-poll.com`.
