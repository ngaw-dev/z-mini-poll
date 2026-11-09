# ADR-0005: Public API Contract (`/api/v1/`) and Idempotent Voting

- **Status:** Accepted (provisional CTO defaults, 2026-11-09)
- **Date:** 2026-11-09
- **Deciders:** CTO (provisional defaults approved for Phase 0 planning)
- **Convention source (2026-11-09, per user direction):** request/response shapes follow the my-brain (mimir) knowledge base — `context/how-to-code/laravel/api-response.md` (ApiResponse trait) and `context/how-to-code/javascript/api-layer.md` (typed API layer). AGENTS.md requires mimir-skill to be loaded before any code work on this contract.

## Context
Next.js public UI consumes Laravel APIs. The PRD required `/api/v1/`, no end-user auth tokens for public voting, server-enforced result visibility, and idempotent same-cookie duplicate handling.

## Decision
All public endpoints are under `/api/v1/`, JSON over HTTPS, no voter login. Responses use the my-brain **ApiResponse** envelope (Laravel `App\Traits\ApiResponse`):

**Success:**
```json
{ "success": true, "message": "All done!", "data": { } }
```

**Error:**
```json
{ "success": false, "message": "human-readable", "errors": { "error": "machine_code" } }
```

- Machine-readable codes (`poll_not_found`, `poll_closed`, `results_not_available`, `validation_failed`, `rate_limited`) are carried in `errors`; HTTP status remains the primary machine contract. Optional `extra` (per the trait) may carry metadata such as `"idempotent": true`.

**Provisional endpoints:**

| Method | Path | Purpose | Auth/cookie | Key responses |
|---|---|---|---|---|
| GET | `/api/v1/polls/{slug}` | Fetch poll for public page | none required; may set/read `mini-poll-id` | 200 `data: poll` (question, options, status, expiry, `has_voted` context; **no results while open and not voted**), 404 |
| POST | `/api/v1/polls/{slug}/votes` | Submit one vote | uses/sets `mini-poll-id` | 201 `data: { voted: true }` counted, 200 `data: { idempotent: true }` duplicate (no count change), 404, 409 closed/expired, 422 validation, 429 |
| GET | `/api/v1/polls/{slug}/results` | Fetch results when allowed | uses `mini-poll-id` for open-poll eligibility | 200 `data: results`, 409 `results_not_available` if open and not voted, 404, 429 |

**Behavioral rules:**
- Vote request body: `{ "option_id": <id> }` (exact field set finalized in Phase 0).
- Server is authoritative for visibility: open poll results never returned before a valid vote for that viewer context (or equivalent server rule), except when closed (results public).
- Same valid `mini-poll-id` + same poll => idempotent; new browser/missing cookie may create a new vote (accepted MVP limitation).
- Standard codes: 400/422 validation, 401/403 authorization/forbidden as applicable, 404 not found, 409 conflict/state (closed/expired/results-not-available), 429 rate limited — mapped to the ApiResponse trait's status messages.

**Frontend consumption (Next.js):** follow `context/how-to-code/javascript/api-layer.md` — typed interfaces (`PollResponse`, `ResultsResponse`, … as `{ success, message, data }`) in `lib/api-polls.ts`; a single shared axios instance (`baseURL: '/api/v1'`, `withCredentials: true`, `Accept: application/json`) so the `mini-poll-id` cookie flows on the same-site proxy (ADR-0001/0003); global response-error interceptor reading `message`/`errors`.

**App-level/privileged tokens:** if used for non-public internal calls, store server-side/config only and never expose to browsers. Not used for public voter endpoints.

## Consequences
- Gives frontend/backend a concrete starting contract; exact field names/pagination/versioning details finalized in Phase 0.
- Result-visibility enforcement depends on reliable cookie handling across the chosen topology (ADR-0001/0003/0007).
- Idempotent response must be safe for client retries.
