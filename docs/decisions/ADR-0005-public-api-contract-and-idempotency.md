# ADR-0005: Public API Contract (`/api/v1/`) and Idempotent Voting

- **Status:** Accepted (provisional CTO defaults, 2026-11-09)
- **Date:** 2026-11-09
- **Deciders:** CTO (provisional defaults approved for Phase 0 planning)

## Context
Next.js public UI consumes Laravel APIs. The PRD required `/api/v1/`, no end-user auth tokens for public voting, server-enforced result visibility, and idempotent same-cookie duplicate handling.

## Decision
All public endpoints are under `/api/v1/`, JSON over HTTPS, no voter login. Errors use a consistent shape:
```json
{ "error": { "code": "string", "message": "human-readable", "details": {} } }
```

**Provisional endpoints:**

| Method | Path | Purpose | Auth/cookie | Key responses |
|---|---|---|---|---|
| GET | `/api/v1/polls/{slug}` | Fetch poll for public page | none required; may set/read `mini-poll-id` | 200 poll (question, options, status, expiry, `has_voted`/context as designed; **no results while open and not voted**), 404 |
| POST | `/api/v1/polls/{slug}/votes` | Submit one vote | uses/sets `mini-poll-id` | 201 counted, 200 idempotent duplicate (`"idempotent": true`, no count change), 404, 409/422 invalid/closed/expired/already-handled state, 429 |
| GET | `/api/v1/polls/{slug}/results` | Fetch results when allowed | uses `mini-poll-id` for open-poll eligibility | 200 results, 403/409 `results_not_available` if open and not voted, 404, 429 |

**Behavioral rules:**
- Vote request body: `{ "option_id": <id> }` (exact schema finalized in Phase 0).
- Server is authoritative for visibility: open poll results never returned before a valid vote for that viewer context (or equivalent server rule), except when closed (results public).
- Same valid `mini-poll-id` + same poll => idempotent; new browser/missing cookie may create a new vote (accepted MVP limitation).
- Standard codes: 400/422 validation, 401/403 authorization/forbidden as applicable, 404 not found, 409 conflict/state (closed/expired), 429 rate limited.

**App-level/privileged tokens:** if used for non-public internal calls, store server-side/config only and never expose to browsers. Not used for public voter endpoints.

## Consequences
- Gives frontend/backend a concrete starting contract; exact field names/pagination/versioning details finalized in Phase 0.
- Result-visibility enforcement depends on reliable cookie handling across the chosen topology (ADR-0001/0003/0007).
- Idempotent response must be safe for client retries.
