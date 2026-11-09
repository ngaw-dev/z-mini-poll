# ADR-0007: CORS/Boundary Defaults and Validation/Presentation Defaults

- **Status:** Accepted (provisional CTO defaults, 2026-11-09)
- **Date:** 2026-11-09
- **Deciders:** CTO (provisional defaults approved for Phase 0 planning)
- **Confirmed items (CTO, 2026-11-09):** same-site `/api/v1` proxy through `mini-poll.com`; case-sensitive duplicate-option-text rejection (remainder of this ADR stays provisional pending Phase 0)

## Context
Separate Next.js/Laravel apps and subdomains require CORS/cookie boundary decisions. Duplicate-option validation and percentage rounding were also marked for confirmation.

## Decision
**CORS / boundaries (provisional):**
- Allow CORS for `/api/v1/*` only from approved web origins: `https://mini-poll.com` (production) and explicit staging/local origins (e.g., `http://localhost:3000`) via config.
- Do not use wildcard origins with credentials.
- **Topology confirmed (CTO, 2026-11-09):** `/api/v1` is proxied through `https://mini-poll.com` (same-site), simplifying cookies/CORS. No separate API origin and no credentialed cross-origin fetch configuration is required.
- Admin (`admin.mini-poll.com`, Filament) is same-origin to its Laravel app surface; no public CORS exposure needed.

**Validation/presentation defaults (provisional):**
- **Duplicate option text:** after trimming whitespace, reject options whose text is identical (case-sensitive comparison) with a clear validation error. **Confirmed by CTO, 2026-11-09.**
- **Percentage rounding:** display percentages to **one decimal place** (e.g., `33.3%`). Visual bars use raw proportions. Rounded values may not sum to exactly 100%; UI should not imply false precision.

## Consequences
- Same-site `/api/v1` proxy minimizes cross-origin cookie complexity (confirmed); remaining work is hosting/reverse-proxy configuration.
- Case-sensitive duplicate rejection is confirmed for MVP; change only via a new product decision.
- Rounding rule gives stable, testable display behavior.
