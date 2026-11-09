# Code Review Workflow

## Entry requirements
- Work is on a dedicated branch.
- Diff and relevant tests are available.
- The issue and acceptance criteria are linked.
- Developer has run relevant checks and recorded results.

## Reviewer checks
1. Correctness against acceptance criteria.
2. Security, authentication and authorisation.
3. Data integrity, migrations and transaction behaviour.
4. API compatibility and validation.
5. Error handling and observability.
6. Performance and scalability.
7. Architecture and maintainability.
8. Tests and regression coverage.
9. Compliance with `mimir-skill`.
10. Ownership boundaries and unintended file changes.

## Outcomes
- APPROVED: no unresolved blocking findings.
- APPROVED WITH COMMENTS: safe to proceed; non-blocking notes remain.
- CHANGES REQUESTED: material findings must be fixed.

The reviewer records findings in the work-tracking system and/or the pull request. The developer addresses findings. The reviewer verifies the corrections. QA follows review approval.

The reviewer must not merge, deploy or change the implementation.
