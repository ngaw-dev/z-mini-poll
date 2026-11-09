# ADR-0004: Rate Limiting, Abuse Controls, and Retention Posture

- **Status:** Accepted (provisional CTO defaults, 2026-11-09)
- **Date:** 2026-11-09
- **Deciders:** CTO (provisional defaults approved for Phase 0 planning)
- **Confirmed items (CTO, 2026-11-09):** rate limit of 100 requests/minute **per client IP** across public `/api/v1/*` endpoints, with 429 + `Retry-After` response; **enforcement layer = Laravel framework rate limiter** (cache-backed), in-app and authoritative — not proxy-only — with **required trusted-proxy configuration** so client IPs resolve correctly behind the reverse proxy (remainder of this ADR stays provisional pending Phase 0)

## Context
The PRD set a 100 requests/minute rate limit but did not define scope. Public no-login endpoints are abuse-prone. IP-hash/privacy design and retention were open.

## Decision
- **Primary limit (confirmed by CTO, 2026-11-09):** **100 requests per minute per client IP** for public `/api/v1/*` endpoints (poll fetch, vote submit, results-as-allowed), applied across the group. **Enforcement is in-app via the Laravel framework rate limiter (cache-backed)** and is authoritative — not proxy-only. The reverse proxy must be configured as a **trusted proxy** (correct `X-Forwarded-For`/header handling) so Laravel resolves the real client IP.
- **Response on limit:** HTTP **429 Too Many Requests** with `Retry-After` header and JSON body using the ApiResponse envelope (ADR-0005):
  ```json
  { "success": false, "message": "Easy there — you're going a bit too fast", "errors": { "error": "rate_limited" } }
  ```
- **Admin/Filament:** rely on Filament/Laravel auth throttling and standard web protections; not covered by the public 100/min `/api/v1/` rule.
- **IP/privacy (provisional):** do not add application-level IP-hash dedup in MVP beyond rate limiting/infrastructure controls. If IP data is processed for abuse control, minimize it and follow infrastructure log retention defaults. No voter PII is collected by design.
- **Retention (provisional):** retain polls, options, aggregate vote records/results, and minimal operational metadata for MVP operation; define formal retention/privacy policy before public launch. Do not store unnecessary PII.

## Consequences
- Simple, predictable baseline; may need tuning for NAT/shared networks.
- Enumeration and vote-spam risk reduced but not eliminated.
- Phase 0 should confirm trusted proxy configuration so per-IP limiting uses real client IP safely.
- Legal/privacy review may tighten retention/IP handling before launch.
