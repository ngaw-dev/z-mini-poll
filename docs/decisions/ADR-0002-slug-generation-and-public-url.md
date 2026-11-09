# ADR-0002: Slug Generation and Canonical Public URL

- **Status:** Accepted (provisional CTO defaults, 2026-11-09)
- **Date:** 2026-11-09
- **Deciders:** CTO (provisional defaults approved for Phase 0 planning)
- **Confirmed items (CTO, 2026-11-09):** slug generated with Laravel **`Str::slug`** from the poll question; **DB existence check** for uniqueness; on collision, append the creation date in **`mmdd`** form. (Supersedes the provisional random base62 default below; remaining details stay provisional.)

## Context
The PRD requires a unique, shareable public poll link and DB-level slug uniqueness, but did not define slug format, entropy/guessability, collision handling, or canonical URL structure. The provisional default was a random base62 slug; the CTO directed a human-readable `Str::slug`-based approach instead.

## Decision
- **Canonical public URL:** `https://mini-poll.com/p/{slug}`.
- **Slug format (confirmed):** generate the slug from the poll question using Laravel **`Str::slug()`** (lowercase, hyphenated, ASCII-safe words), e.g. `where-should-we-eat`.
- **Collision handling (confirmed):** check the DB for an existing slug. If it exists, append the creation date in **`mmdd`** form: `where-should-we-eat-1109`. If that also collides, append a short random suffix and retry (bounded retries; provisional default) until unique.
- **Uniqueness/integrity:** enforce a **unique DB index/constraint** on `slug` as the backstop; the DB check above is the normal path.
- **Mutability:** slugs are **immutable** after creation in MVP.
- **Guessability posture:** slugs are derived from question text and are **guessable/enumerable**; the share link is a public bearer link — do not treat slug secrecy as authorization. Rate limiting (ADR-0004, 100 req/min per IP) mitigates enumeration. Accepted for casual MVP use; revisit for sensitive polls in v1.
- **Edge cases (provisional):** if `Str::slug()` yields an empty string (question with no slug-able characters) or an over-long slug from a 200-char question, fall back to a truncated slug plus a random suffix (bounded length); exact truncation limit set in implementation.
- **Invalid slug:** public API/page returns a clear **404 Not Found** state.

## Consequences
- Human-readable, self-describing share links.
- Lower entropy than random IDs: enumeration is easier; mitigated (not eliminated) by rate limiting. Privacy-sensitive questions leak their topic into the URL.
- Same-question polls created on the same day can still collide after the `mmdd` suffix; the random-suffix fallback covers this.
- Copy-link uses the absolute canonical URL.
- Admin UI and API responses should expose the canonical URL/slug consistently.
