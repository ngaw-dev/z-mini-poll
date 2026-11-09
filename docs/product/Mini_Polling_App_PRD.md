# Product Requirements Document — Mini Polling App

| Field | Value |
|---|---|
| Document Owner | Product |
| Status | Draft for review |
| Target Release | MVP — 6 weeks |
| Platform | Web app (responsive, mobile + desktop) |
| Auth | Email + password |

---

## 1. Overview

The Mini Polling App is a lightweight web application that lets registered users create simple polls, share them via a link, and collect votes from anyone who opens the link. Each voter can vote only once per poll, and results update in real time as votes come in.

The product targets small teams, communities, event organizers, and casual users who need a fast, no-frills way to make group decisions — "where to eat", "which date works", "which feature to build next" — without the overhead of a full survey tool.

### Problem Statement
Existing polling tools (Google Forms, Doodle, Straw Poll) are either too heavy, require too many clicks, or feel generic. There is room for a clean, fast, link-shareable polling app with a modern UX and instant result visibility.

### Why Now
A focused MVP can be shipped in 4–6 weeks with a single developer. The product also serves as a strong portfolio/reference project for the team.

---

## 2. Goals & Non-Goals

### Goals
- Let a registered user create a poll with a question and 2–6 options in under 30 seconds.
- Let anyone with the share link vote exactly once per poll.
- Show real-time, accurate results to the poll creator and to voters after they vote.
- Ship a clean, responsive UI that works on mobile and desktop.

### Non-Goals (for MVP)
- No image or rich-media polls.
- No public poll discovery / marketplace.
- No multi-vote (approval voting, ranked choice) — single choice only.
- No anonymous creator mode — poll creators must be logged in.
- No SSO / social login — email + password only.

---

## 3. Target Users

### Persona A — "The Organizer" (creator)
- Example: A team lead running a weekly retro, a friend group picking a restaurant.
- Needs: Fast poll creation, a shareable link, clear results.
- Pain: Existing tools are slow, ad-heavy, or require sign-up just to vote.

### Persona B — "The Voter" (participant)
- Example: Anyone who receives a poll link in chat or email.
- Needs: One-tap vote, see results, no friction.
- Pain: Forced account creation, confusing UI, unclear whether their vote was counted.

---

## 4. User Stories & Acceptance Criteria

### Epic 1 — Account Management

**US 1.1 — Register**
As a new user, I want to sign up with email and password so I can create polls.
- Acceptance:
  - Email must be unique and valid format.
  - Password ≥ 8 characters.
  - On success, user is logged in and lands on the dashboard.
  - On failure, clear inline errors are shown.

**US 1.2 — Log in / Log out**
As a returning user, I want to log in and out so I can access my polls.
- Acceptance:
  - Wrong credentials show a generic "invalid email or password" error.
  - Logout clears the session and redirects to the login page.

### Epic 2 — Poll Creation

**US 2.1 — Create a poll**
As a logged-in user, I want to create a poll with a question and 2–6 options.
- Acceptance:
  - Question is required, 5–200 characters.
  - Minimum 2 options, maximum 6 options.
  - Each option text is required, 1–80 characters.
  - On submit, a unique shareable link is generated immediately.
  - Creator lands on the poll's results page after creation.

**US 2.2 — Optional poll expiry**
As a creator, I want to optionally set a poll to expire at the end of a chosen day.
- Acceptance:
  - Creator can pick "no expiry" (default) or a specific calendar date.
  - When a date is chosen, the poll expires at **midnight (23:59:59) in the voter's local time** on that date.
  - After expiry, the poll is locked — no new votes accepted, but results remain viewable.
  - The poll page shows the expiry date to voters so they know how long they have to vote.

### Epic 3 — Voting

**US 3.1 — Vote on a poll**
As anyone with the link, I want to cast a single vote.
- Acceptance:
  - Voter must select exactly one option.
  - Submitting the vote records it and shows the results page.
  - Voter cannot vote again on the same poll from the same browser.
- Enforcement:
  - Backend: vote deduplication by a combination of voter token (cookie-based, anonymous users) and IP hash.
  - If a logged-in user votes, their user ID is recorded to prevent re-voting from another browser.

**US 3.2 — See results**
As a voter or creator, I want to see current vote counts and percentages.
- Acceptance:
  - Results show each option with vote count, percentage, and a visual bar.
  - Total vote count is displayed.
  - Results update without page refresh (polling every 5 seconds in MVP; WebSockets in v2).

### Epic 4 — Poll Management

**US 4.1 — View my polls**
As a creator, I want to see all polls I have created.
- Acceptance:
  - Dashboard lists polls sorted by most recently created.
  - Each row shows question, total votes, status (open/expired), and a copy-link button.

**US 4.2 — Close a poll manually**
As a creator, I want to close a poll before its expiry.
- Acceptance:
  - A "Close poll" button appears on the creator's own poll page.
  - After closing, no new votes are accepted; results remain visible.

---

## 5. Functional Requirements

### Authentication
- Email + password registration and login.
- Sessions/tokens managed securely; storage mechanism to be decided by engineering (httpOnly cookies or local storage) pending a security review.

### Poll Lifecycle
- States: **Open** → **Closed** (by expiry or manual close).
- A closed poll is read-only but viewable forever via the same link.

### Vote Integrity
- Each browser (anonymous voter) gets one vote per poll, enforced by a voter token cookie.
- Logged-in users are bound by user ID.
- MVP uses IP hash + voter token as a soft dedup; not bulletproof, but sufficient for casual use.
- Expiry is evaluated at midnight local time on the chosen date, so the poll remains open for the full final day.

### Sharing
- Each poll gets a public share URL: `/poll/{slug-or-id}`.
- No authentication required to view or vote on a poll via this URL.

### Responsive UI
- Mobile-first; works down to 360px wide.
- Clean, modern component library (specific choice deferred to engineering).

---

## 6. Success Metrics

| Metric | MVP Target | Measurement |
|---|---|---|
| Time to create a poll (creator, from "New poll" click to share link) | < 30 seconds | Session recording / manual test |
| Vote conversion rate (% of link openers who vote) | ≥ 70% | Backend event logging |
| Polls created per active user (week 4) | ≥ 3 | DB query |
| Average options per poll | 3–4 | DB query (sanity check on UX) |
| Page load on poll share page (mobile, 4G) | < 2 seconds | Lighthouse / RUM |
| Vote re-submit attempts blocked | 100% | Backend dedup counter |
| 7-day returning creator rate | ≥ 25% | Auth log analysis |

These metrics will be reviewed 2 weeks post-launch and used to prioritize v1 improvements.

---

## 7. Milestones

### Phase 0 — Setup (Week 1)
- Backend project scaffold, auth, database migrations.
- Frontend project scaffold, styling baseline, data-fetching setup.
- CORS + auth flow working end-to-end (register, login, logout).

### Phase 1 — MVP Core (Weeks 2–4)
- Poll create/read endpoints + UI.
- Public vote endpoint + dedup logic.
- Results page with auto-refresh.
- Dashboard for creators.
- Share link generation and copy-to-clipboard.

### Phase 2 — Polish & Launch (Weeks 5–6)
- Poll expiry + manual close.
- Mobile responsive QA pass.
- Empty states, loading states, error states.
- Basic analytics events (poll created, vote cast, link opened).
- Deploy to staging, then production.
- Internal dogfooding for 3 days before public launch.

### Phase 3 — Post-Launch (Week 7+)
- Collect feedback, monitor metrics.
- Triage bugs, ship quick wins.
- Plan v1 scope based on data.

---

## 8. Future Roadmap (Post-MVP)

Ordered by likely impact, not by commitment:

1. **Live results via push updates** — Replace periodic refresh with real-time push (e.g. WebSockets) for instant updates. High user-facing value.
2. **Image polls** — Allow an image per option. Great for "which logo", "which outfit" use cases.
3. **Anonymous creator mode** — Create polls without an account; polls expire in 24h.
4. **Public poll gallery** — Browse featured/trending polls. Requires moderation tooling first.
5. **Approval / ranked-choice voting** — Power-user feature; expands the product beyond simple polls.
6. **Embed widgets** — Embeddable poll iframe for blogs and newsletters.
7. **Export results** — CSV/JSON export for creators.
8. **SSO / social login** — Google, GitHub login to reduce sign-up friction.
9. **Custom branding** — Let creators pick colors / add a logo to their poll page.
10. **Webhook on poll close** — For integrations (Slack notification when a poll closes).

---

## 9. Risks & Open Questions

| Risk / Question | Status | Mitigation |
|---|---|---|
| Vote dedup is soft — determined users can clear cookies + change IP to vote twice. | Accepted for MVP | Acceptable for casual use cases. Hard enforcement (account-required voting) deferred to v1. |
| Should voters be required to log in? | Open | Lean toward no — friction kills conversion. Revisit based on creator feedback. |
| Spam / abuse of public polls | Open | Rate-limit vote endpoint by IP in MVP. Add report button in v1. |
| Data retention — how long do we keep polls? | Open | MVP: forever. Add a "delete my poll" feature in v1. |
| Branding — name, domain, logo | Open | Needs product/design input before launch. |

---

## 10. Out of Scope (Explicit)

The following are explicitly excluded from MVP and will not be built even if time allows:

- Admin panel / moderation tools.
- Paid plans or monetization.
- Email notifications (voters or creators).
- Multi-language support.
- Accessibility audit beyond basic semantic HTML and keyboard nav.
- Performance optimization beyond Lighthouse score of 90+ on key pages.

---

*End of PRD — for review and feedback before development kicks off.*
