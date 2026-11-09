# Product Requirements Document — Mini Polling App

| Field | Value |
|---|---|
| Document Owner | Product |
| Status | **Approved (Product Owner) — 2026-11-09** |
| Based on | `docs/product/Mini_Polling_App_PRD.md` + captured PO/architecture decisions in `PRD_Review_Decisions_Addendum.md` (2026-11-09) |
| Target Release | MVP — 6 weeks (subject to Phase 0 architecture sign-off) |
| Public platform | Web app, responsive (mobile + desktop), **Next.js** at `https://mini-poll.com` |
| Admin platform | **Laravel + Filament** dashboard at `https://admin.mini-poll.com` |
| Backend | **Laravel** (APIs, auth, poll/vote logic, expiry, slug/rate limiting, Filament admin) |
| Voter auth | **No login required** |
| Poll creation | **Single admin only** (Filament admin dashboard); no public/anonymous creator mode |

---

## 1. Overview

The Mini Polling App is a lightweight web application that lets a **single administrator** create simple polls from an admin dashboard, share them via a public link, and collect votes from anyone who opens the link. Voters do not need an account. Each voter submission is constrained by MVP deduplication rules (cookie-based; a new browser or missing cookie may submit again — accepted limitation). Results are hidden before voting on open polls and become visible after a valid vote; closed polls show results to anyone. Results refresh automatically.

The product targets small teams, communities, event organizers, and casual audiences who need a fast, no-frills way to make group decisions — "where to eat", "which date works", "which feature to build next".

### Problem Statement
Existing polling tools can be heavy, ad-heavy, or add unnecessary friction. There is room for a clean, fast, link-shareable polling app with a simple admin-operated publishing model and clear public voting UX.

### MVP Operating Model
MVP uses an **admin-operated** model: only one admin can create/manage polls. This is intentionally narrower than a self-serve creator platform.

---

## 2. Goals & Non-Goals

### Goals
- Let the admin create a poll with a question and 2–6 options in under 30 seconds.
- Let anyone with the public share link vote without logging in.
- Prevent accidental duplicate counting from the same browser/session where the `mini-poll-id` voter cookie is present (idempotent duplicate handling).
- Hide results before voting on open polls; show accurate results after a valid vote and for closed polls.
- Enforce poll expiry server-side using `APP_TIMEZONE` and a clear end-of-day boundary.
- Ship a clean, responsive public UI (Next.js) and a secure admin dashboard (Filament).

### Non-Goals (MVP)
- No public/self-serve poll creation or user registration for creators.
- No anonymous creator mode.
- No multi-admin roles/permissions UI (single admin only).
- No voter accounts/login.
- No image or rich-media polls.
- No public poll discovery / marketplace.
- No multi-vote (approval voting, ranked choice) — single choice only.
- No SSO/social login.
- No email notifications.
- No admin audit trail beyond standard application/infrastructure logs. (Filament/application logging may be used operationally.)

---

## 3. Target Users

### Persona A — "The Admin Publisher"
- Example: Amol/team member operating Mini Poll for events or team decisions.
- Needs: Fast poll creation, shareable slug/link, live/after-vote results, ability to close polls, secure access.
- Pain: Public creator tools add complexity; MVP needs controlled publishing.

### Persona B — "The Voter" (public participant)
- Example: Anyone who receives a poll link in chat or email.
- Needs: One-tap vote, clear confirmation, see results after voting, no account friction.
- Pain: Forced account creation, confusing UI, unclear whether vote counted, seeing results too early.

---

## 4. User Stories & Acceptance Criteria

## Epic 1 — Admin Access (Single Admin)

**US 1.1 — Admin access (single admin, CLI-provisioned)**
As the single admin, I want to access `admin.mini-poll.com` so I can manage polls.
- Acceptance:
  - The single admin account is created/provisioned via **command line** (no public registration, no admin-creation API required for MVP).
  - Admin authentication and dashboard access are managed via **Filament** on `admin.mini-poll.com`; Filament built-in auth is used.
  - Only the provisioned admin can access Filament/admin routes. Invalid credentials produce a generic error.
  - Authenticated admin lands on the poll management dashboard.
  - Public users cannot access admin routes.
  - Filament/application logging is acceptable for operational visibility; no separate admin audit trail is required beyond standard application/infrastructure logs.
  - Password reset/recovery posture, login throttling, and session hardening are decided in **ADR-0006**.

**US 1.2 — Admin can sign out**
- Acceptance:
  - Sign out invalidates the admin session according to the approved auth design.
  - After sign out, admin is redirected to admin login and cannot access protected pages without re-authenticating.

## Epic 2 — Poll Creation & Lifecycle (Admin)

**US 2.1 — Create a poll**
As the admin, I want to create a poll with a question and 2–6 options.
- Acceptance:
  - Question is required; after trimming whitespace, length is 5–200 characters.
  - Minimum 2 options, maximum 6 options.
  - Each option is required; after trimming whitespace, length is 1–80 characters.
  - Duplicate option text after trimming is rejected with a clear validation error (default per **ADR-0007**).
  - Option order is preserved as entered.
  - **Filament manages form validation** for admin create/edit flows; backend/database enforce business rules and integrity.
  - On submit, the system generates a unique poll identifier and slug; **slug uniqueness is enforced at DB level** (format/entropy/collision per **ADR-0002**).
  - Admin sees the created poll with its canonical public share URL: `https://mini-poll.com/p/{slug}` (proposed).
  - Poll is created in **Open** state unless/until expired or manually closed.

**US 2.2 — Optional poll expiry**
As the admin, I want to optionally set a poll to expire at the end of a chosen calendar date.
- Acceptance:
  - Admin can choose **No expiry** (default) or a specific calendar date.
  - If a date is selected, expiry is interpreted in `APP_TIMEZONE`.
  - The system stores canonical `expires_at` representing `YYYY-MM-DD@23:59:59` in `APP_TIMEZONE` (recommended: also persisted/comparable as UTC instant) and enforces it server-side.
  - New votes are rejected when current server time is after `expires_at`.
  - After expiry, poll state is **Closed** (expired); no new votes accepted; results remain viewable.
  - Public poll page and admin view display the expiry date/time and timezone clearly.

## Epic 3 — Public Voting (No Login)

**US 3.1 — View and vote on a poll**
As anyone with the public link, I want to vote once from my current browser/session.
- Acceptance:
  - Public page is accessible without login at the canonical share URL on `mini-poll.com`.
  - If poll is **Open** and viewer has not submitted a valid vote in the current browser/session context, UI shows question and selectable options; **results are not shown before voting**.
  - Voter must select exactly one option.
  - Submitting records the vote and shows the results view with a clear success/recorded state.
  - Public API/UI handles validation errors clearly (missing option, poll closed/expired, invalid poll).

**US 3.2 — Deduplication and revote behavior (MVP)**
- Acceptance:
  - Backend issues/uses voter token cookie **`mini-poll-id`** for anonymous dedup: **HttpOnly, 24-hour expiry**. **Do not use localStorage** for voter dedup.
  - Cookie attributes for `mini-poll.com` are defined in **ADR-0003**.
  - If the same valid `mini-poll-id` token submits again for the same poll, the request is handled **idempotently**: no additional vote is counted; API returns an appropriate non-counting response/state.
  - If the request is from a **new browser** or **no `mini-poll-id` cookie is present**, a new vote may be accepted (known MVP limitation, not a defect).
  - Vote recording/count updates are atomic under concurrent requests.
  - §6 success metrics must not claim 100% prevention of all possible re-votes; they may claim correct idempotent handling for defined dedup cases.

**US 3.3 — See results**
As a voter (after voting) or any viewer of a closed poll, I want to see current results.
- Acceptance:
  - Results show each option with vote count, percentage, and a visual bar.
  - Total vote count is displayed.
  - Percentages are calculated consistently and displayed with a defined rounding rule (one decimal place per **ADR-0007**).
  - Results refresh/presentation behavior is handled by the UI (Next.js public app) while the poll is open and results are visible (proposed: every ~5 seconds, with reasonable error handling/backoff). After close, polling stops or becomes unnecessary.
  - Open polls do not expose results to non-voters before voting.

## Epic 4 — Poll Management (Admin Dashboard)

**US 4.1 — View polls in admin dashboard**
As the admin, I want to see and manage polls I created.
- Acceptance:
  - **Filament handles** the admin poll list/dashboard UX.
  - Dashboard should provide at minimum: question, slug/public link, total votes, status (**Open**, **Closed — expired**, **Closed — manual**), expiry (if any), and created date.
  - Admin can copy the canonical public share URL.
  - Empty state, pagination/search/filter behavior are managed by Filament/default implementation unless product specifies otherwise.

**US 4.2 — Close a poll manually**
As the admin, I want to close a poll before expiry.
- Acceptance:
  - Admin can manually close an Open poll from the dashboard/poll detail.
  - After manual close, poll becomes **Closed — manual**; new votes are rejected; results remain viewable.
  - Closing an already closed poll is a safe no-op or clearly handled.

---

## 5. Functional Requirements

### Domains & Topology
- Public: `https://mini-poll.com` — Next.js.
- Admin: `https://admin.mini-poll.com` — Laravel + Filament.
- Backend: Laravel application/services providing public APIs and admin/Filament functionality.
- Deployment target includes a **Docker image** for the application stack; repo layout, service split, and deployment topology are decided in **ADR-0001**.

### Authentication & Authorization
- No authentication for public voting/viewing.
- Single admin account only; admin created via command line; no admin-creation API required. Filament provides admin auth/dashboard access.
- All admin routes require authenticated admin access.
- Password reset/recovery posture, session hardening, CSRF details, login throttling, and secrets/config management are decided in **ADR-0006**.

### Public Share URL & Slug
- Canonical public URL (proposed): `https://mini-poll.com/p/{slug}`.
- Slug/identifier must be unique; **uniqueness enforced at DB level**. Generation rules, length/charset, collision handling, mutability, and guessability posture are decided in **ADR-0002**.
- Invalid slug returns a clear not-found state.
- Copy action copies the absolute canonical URL.

### Poll Lifecycle
- States: **Open** → **Closed**.
- Closed reasons: **expired** or **manual**.
- Closed polls are read-only for voting and remain viewable.

### Expiry
- Optional expiry date selected by admin.
- Interpreted in `APP_TIMEZONE`; closes at `23:59:59` in `APP_TIMEZONE` on selected date.
- Server is source of truth; canonical `expires_at` stored/enforced server-side.
- UI shows expiry with timezone.

### Vote Integrity (MVP)
- No voter login.
- Cookie-based anonymous voter token used for dedup/idempotency.
- New browser/missing cookie may submit again — accepted MVP limitation.
- Duplicate same-token submission for same poll is idempotent (no double count).
- Atomic vote recording under concurrency.
- IP/rate-limit abuse controls, IP-hash/privacy design, and retention posture are decided in **ADR-0004**.

### API Contract
- Public-facing API routes are under **`/api/v1/`**.
- **Rate limit:** 100 requests per minute (scope and response shape per **ADR-0004**).
- **Public endpoints** (e.g., `/api/v1/polls/{slug}`, vote submit, results as allowed): accessible without voter login; protected by rate limiting, validation, `mini-poll-id` dedup/idempotency, and server-side visibility rules. **No end-user auth tokens required/exposed** for public voting.
- **App-level/backend or privileged API calls** (if any): auth tokens are stored **server-side/config only and never exposed to browsers**.
- **Admin management:** handled via Filament web/admin auth on `admin.mini-poll.com`, not a public admin API.
- Endpoint list, request/response schemas, validation errors, 401/403/404/409/429 behaviors, idempotency responses, and token scope are decided provisionally in **ADR-0005**.

### Responsive UI & Accessibility
- Public UI mobile-first; works down to 360px wide.
- **No accessibility (a11y) requirements at this time (MVP).**

---

## 6. Success Metrics (Proposed)

| Metric | MVP Target | Measurement | Notes |
|---|---|---|---|
| Time for admin to create a poll (create action to share URL visible) | < 30 seconds | Manual timed test script (N runs) | Admin-operated flow; replace prior “registered creator” metric. |
| Vote completion rate (public poll page load to valid vote submitted) | ≥ 70% | Server/app event logging | Define bot/filter rules; do not count duplicate idempotent retries as extra completions. |
| Polls created during MVP evaluation window | No target (MVP) | DB query (optional) | Single-admin operating model; not a growth/activation target for MVP. |
| Average options per poll | 3–4 | DB query | Sanity check. |
| Public poll page load (mobile, agreed test method) | No LCP target (MVP) | N/A unless later added | Performance target deferred. |
| Duplicate submissions with same voter token | 100% handled idempotently (no double count) | Automated/integration tests + logs | Does **not** claim prevention of new-browser/no-cookie re-votes. |
| Admin can close poll and block new votes | 100% in acceptance tests | QA | Critical control. |

Review metrics 2 weeks post-launch and revise for the actual operating model.

---

## 7. Milestones

### Phase 0 — Architecture & Setup (Week 1)
- **CTO architecture sign-off:** topology, Laravel/Next.js boundaries, Filament admin hosting, Docker image/deployment split, `/api/v1/` contract, auth/CSRF/session, dedup cookie/local-storage clarification (`mini-poll-id`), `APP_TIMEZONE`/expiry storage, slug strategy, rate limiting (100/min scope), app-level token scope, secrets/config.
- Repositories/projects/environments scaffolded.
- Database migrations and single-admin provisioning approach.
- Docker/local dev workflow, CORS/auth baseline, and `/api/v1/` routing working end-to-end.

**Exit criteria:** approved architecture notes/ADRs, API contract draft, admin can access Filament locally/staging, public Next.js can fetch a test endpoint.

### Phase 1 — MVP Core (Weeks 2–4)
- Laravel `/api/v1/` public APIs: poll by slug, vote submit (idempotent/atomic, `mini-poll-id` dedup), results (visibility rules enforced).
- Filament admin: CLI-provisioned single admin, create/list/manage polls, expiry, manual close, copy share URL (Filament-managed validation/UI where applicable).
- Next.js public pages: poll view, vote flow, post-vote/closed results, 5s refresh behavior.
- Slug/share URL generation and copy-to-clipboard.
- Validation, error states, empty/loading states.

### Phase 2 — Polish & Launch (Weeks 5–6)
- Expiry edge-case QA (`APP_TIMEZONE`, 23:59:59 boundary).
- Mobile responsive QA pass (minimum 360px; browsers/devices agreed with QA).
- Rate limiting (100 requests/minute, scope as approved) and basic abuse protections.
- Minimal analytics/event logging feasible in MVP (as approved).
- Deploy staging, then production; DNS/TLS for `mini-poll.com` and `admin.mini-poll.com`.
- Internal dogfooding (3 days) with bug-fix buffer before public launch.

### Phase 3 — Post-Launch (Week 7+)
- Collect feedback and monitor metrics.
- Triage bugs and ship quick wins.
- Revisit self-serve creators/multi-admin/public features only as explicit v1+ scope.

---

## 8. Future Roadmap (Post-MVP, not committed)

1. Real-time push results (WebSockets/SSE) instead of polling.
2. Multi-admin roles/permissions and audit trail.
3. Self-serve creator model (major product/security redesign).
4. Image polls.
5. Stronger anti-abuse / verified voting options.
6. Embed widgets.
7. Export results (CSV/JSON).
8. Custom branding.
9. Webhook/integration on poll close.

---

## 9. Risks & Open Questions

| Risk / Question | Status | Proposed mitigation / owner |
|---|---|---|
| Single-admin credential compromise gives full poll control. | Open | Strong password/hash, HTTPS, throttling, minimal attack surface, secure provisioning/recovery. CTO/Surya. |
| `mini-poll-id` cookie dedup allows new-browser/no-cookie re-votes. | Accepted MVP limitation | Explicit behavior; idempotent same-cookie handling; 100/min rate limit. PO/CTO. |
| `APP_TIMEZONE`/DST/server-time configuration errors can close polls unexpectedly. | Open | Use explicit configured timezone, store canonical expiry, add timezone-aware tests. CTO/Surya/QA. |
| Next.js + Laravel + Filament across subdomains creates CORS/CSRF/cookie complexity. | Open | Resolve in Phase 0 architecture; document contracts and security controls. CTO. |
| Public poll slug/share URL may be guessable/enumerable depending on format. | Open | Choose slug/ID entropy policy; DB-unique slug; 100/min rate limiting; decide privacy posture. CTO/PO. |
| Abuse/spam of public vote endpoint. | Open | Rate-limit by IP/token/poll; monitoring; consider CAPTCHA/post-MVP controls only if approved. Surya/CTO. |
| Data retention/privacy (polls, votes, tokens, logs). | Open | Define retention and privacy posture before launch; avoid storing unnecessary PII. PO/CTO. |
| Name/domain/branding finalization. | Open | Product/design input before launch. PO/Charu. |

---

## 10. Out of Scope (Explicit)

- Public creator registration or self-serve poll creation.
- Multiple admins / RBAC UI.
- Voter login/accounts.
- Admin moderation panel beyond MVP poll management.
- Paid plans/monetization.
- Email notifications.
- Multi-language/i18n.
- Full accessibility audit (WCAG certification) beyond agreed MVP baseline.
- Advanced anti-fraud/identity verification.
- Image/media polls, ranked-choice/approval voting, embeds, exports, webhooks.

---

## 11. Acceptance Checklist Before Development

- [x] PO approves this revised PRD scope (admin-only, single admin, domains, voter rules).
- [ ] CTO ratifies provisional architecture defaults (ADRs 0001–0007): repo/Docker topology, `/api/v1/` contract, Filament auth, `mini-poll-id` cookie, app-level token scope, `APP_TIMEZONE` expiry, slug format, 100/min rate-limit scope, secrets.
- [ ] Prashant/Surya agree frontend-backend API contract and error shapes.
- [ ] Charu provides design direction for public poll UI and/or Filament customization needs (if any).
- [ ] Mohan converts acceptance criteria into executable test cases (including dedup idempotency, expiry boundary, results visibility, rate limits).

