# ADR-0007: CORS/Boundary Defaults and Validation/Presentation Defaults

- **Status:** Accepted (provisional CTO defaults, 2026-11-09)
- **Date:** 2026-11-09
- **Deciders:** CTO (provisional defaults approved for Phase 0 planning)

## Context
Separate Next.js/Laravel apps and subdomains require CORS/cookie boundary decisions. Duplicate-option validation and percentage rounding were also marked for confirmation.

## Decision
**CORS / boundaries (provisional):**
- Allow CORS for `/api/v1/*` only from approved web origins: `https://mini-poll.com` (production) and explicit staging/local origins (e.g., `http://localhost:3000`) via config.
- Do not use wildcard origins with credentials.
- Because `mini-poll-id` is cookie-based, Phase 0 must confirm the final topology: **preferred default** is to proxy `/api/v1` through `https://mini-poll.com` (same-site) to simplify cookies/CORS; if using a separate API origin, configure credentialed CORS/fetch (`credentials: include`) and cookie attributes carefully.
- Admin (`admin.mini-poll.com`, Filament) is same-origin to its Laravel app surface; no public CORS exposure needed.

**Validation/presentation defaults (provisional):**
- **Duplicate option text:** after trimming whitespace, reject options whose text is identical (case-sensitive comparison) with a clear validation error. **[PO confirm default accepted provisionally; revisit if case-insensitive desired]**
- **Percentage rounding:** display percentages to **one decimal place** (e.g., `33.3%`). Visual bars use raw proportions. Rounded values may not sum to exactly 100%; UI should not imply false precision.

## Consequences
- Same-site `/api/v1` proxy minimizes cross-origin cookie complexity; confirm with hosting/reverse-proxy plan.
- Case-sensitive duplicate rejection is simple; product may later prefer case-insensitive normalization.
- Rounding rule gives stable, testable display behavior.
