# ADR-0001: Repository Layout and Docker Deployment Topology

- **Status:** Accepted (provisional CTO defaults, 2026-11-09)
- **Date:** 2026-11-09
- **Deciders:** CTO (provisional defaults approved for Phase 0 planning)

## Context
The PRD requires separate frontend and backend applications: Next.js public UI (`mini-poll.com`), Laravel backend APIs, and Laravel+Filament admin (`admin.mini-poll.com`). The repo is currently a single repository and deployment topology (service split, reverse proxy, local dev) was undecided.

## Decision
Adopt a **monorepo** with clear application boundaries:

- `backend/` — Laravel application: `/api/v1/*` public APIs, vote/expiry/dedup logic, migrations, and Filament admin panel.
- `frontend/` — Next.js application: public poll pages on `mini-poll.com`.
- `docker/` and/or root compose files — local development and deployment wiring.

**Deployment/default runtime topology (provisional):**
- **backend service** (Laravel/PHP-FPM + web server as appropriate): serves Filament/admin on `admin.mini-poll.com` and public API on an API origin/path agreed in Phase 0 (prefer `https://mini-poll.com/api/v1` via reverse proxy, or `https://api.mini-poll.com/v1` with CORS).
- **frontend service** (Next.js): serves `https://mini-poll.com` public UI.
- **reverse proxy** (e.g., Nginx/Caddy/Traefik): TLS termination, host/path routing, and staging parity.
- **Docker Compose** for local development; production image/deployment split finalized in Phase 0 (single compose stack, separate images, or platform-specific manifests).

Staging must use equivalent hostnames/routing conventions.

## Consequences
- Clear ownership boundaries: backend/Surya, frontend/Prashant (per AGENTS.md), with explicit API contracts.
- Requires Phase 0 confirmation of exact API origin (`/api/v1` proxied under `mini-poll.com` vs dedicated API host) because it affects CORS, cookies, and Next.js server/client fetching.
- Filament remains in the Laravel app; no separate Next.js admin.
- Docker images/build contexts must be defined per app to keep CI/CD predictable.
