# ADR-0002: Slug Generation and Canonical Public URL

- **Status:** Accepted (provisional CTO defaults, 2026-11-09)
- **Date:** 2026-11-09
- **Deciders:** CTO (provisional defaults approved for Phase 0 planning)

## Context
The PRD requires a unique, shareable public poll link and DB-level slug uniqueness, but did not define slug format, entropy/guessability, collision handling, or canonical URL structure.

## Decision
- **Canonical public URL:** `https://mini-poll.com/p/{slug}`.
- **Slug format:** 10-character, URL-safe **base62** slug (`[A-Za-z0-9]`), generated server-side using a cryptographically secure random generator. (~59 bits of entropy; not derived from the poll question).
- **Uniqueness/integrity:** enforce a **unique DB index/constraint** on `slug`. On the rare collision, regenerate and retry (bounded retries, then fail create with an operational error).
- **Mutability:** slugs are **immutable** after creation in MVP.
- **Guessability posture:** slugs are unguessable enough for casual sharing but are public bearer links; anyone with the link can access the poll. Do not treat slug secrecy as authorization. Rate limiting (ADR-0004) mitigates enumeration.
- **Invalid slug:** public API/page returns a clear **404 Not Found** state.

## Consequences
- Copy-link uses the absolute canonical URL.
- Short, readable links without leaking poll content.
- Admin UI and API responses should expose the canonical URL/slug consistently.
- Future custom/human slugs would require a migration/product decision (not MVP).
