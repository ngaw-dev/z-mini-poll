# ADR-0003: Voter Deduplication Cookie (`mini-poll-id`)

- **Status:** Accepted (provisional CTO defaults, 2026-11-09)
- **Date:** 2026-11-09
- **Deciders:** CTO (provisional defaults approved for Phase 0 planning)
- **Confirmed items (CTO, 2026-11-09):** full cookie attribute set — `mini-poll-id` = `HttpOnly; Secure; SameSite=Lax; Path=/`, host-only domain (no `Domain` attribute), 24-hour `Max-Age`, no localStorage. Host-only scope is consistent with the confirmed same-site `/api/v1` proxy (ADR-0001/0007).

## Context
MVP uses cookie-based anonymous dedup. A new browser or missing cookie may submit another vote (accepted limitation). The PRD captured `mini-poll-id`, HttpOnly, 24h expiry, and explicitly disallowed localStorage.

## Decision
- **Cookie name:** `mini-poll-id`.
- **Purpose:** anonymous voter token for same-browser/session dedup and idempotent duplicate handling per poll.
- **Value:** opaque, cryptographically random token (sufficient entropy; do not encode PII). Store/hash server-side as needed for dedup checks (implementation detail; avoid storing unnecessary raw identifiers).
- **Lifetime:** 24 hours (`Max-Age=86400`).
- **Attributes (confirmed by CTO, 2026-11-09):** `HttpOnly; Secure; SameSite=Lax; Path=/`. **Domain:** host-only (omit `Domain` attribute) — consistent with the confirmed same-site `/api/v1` proxy under `mini-poll.com`.
- **Client storage:** **no localStorage/sessionStorage** for voter dedup.
- **Behavior:** same valid token + same poll => idempotent, no double count. Missing cookie/new browser => may create a new vote (accepted MVP limitation).

## Consequences
- `Secure` requires HTTPS in all non-local environments.
- If the public API is served from a different origin than the Next.js page, Phase 0 must confirm cookie issuance/credentialed CORS/fetch behavior (ADR-0001/ADR-0007).
- 24h expiry means dedup is time-bounded by cookie lifetime, consistent with MVP’s casual-use posture.
- QA must test idempotent repeats, missing-cookie revote path, and expiry boundary behavior.
