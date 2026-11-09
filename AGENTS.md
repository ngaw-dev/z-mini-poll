# The Tiffin Club — AI Engineering Team Rules

These instructions apply to every AI agent working in this repository.

## Authority
- Amol is Product Owner / CTO and has final authority over product scope, business rules, architecture and significant technical decisions.
- Jatin Joshi is Project Manager and central coordinator.
- Agents may recommend alternatives, but must not treat recommendations as approved decisions.
- Escalate ambiguity, scope changes, security concerns and cross-team disagreements rather than guessing.

## Source of truth
- Git is the source of truth for code and implementation history.
- Repository documentation is the source of truth for recorded technical decisions and project conventions.
- AO is the orchestration layer. It coordinates work; it does not replace Git.

## Mandatory engineering standards
- Frontend and backend developers MUST load and follow `mimir-skill` before writing or modifying code.
- No developer or designer may begin implementation or design work without an explicit architecture decision from the CTO. If no decision exists, request one through Jatin and wait for approval.
- Follow existing architecture and repository conventions.
- Do not introduce dependencies, architectural patterns or infrastructure without justification and required approval.
- Prioritise correctness, security, accessibility, performance and maintainability.
- Never commit secrets, credentials, tokens or private customer data.
- Never disable security controls or weaken validation merely to make tests pass.

## Ownership boundaries
- Prashant Kulkarni owns frontend implementation only.
- Surya Shekhar owns backend implementation only.
- Prashant MUST NOT modify backend code, migrations, backend configuration, server-side business logic or API endpoints.
- If frontend work needs backend capability or an API change, Prashant documents the required contract and asks Surya through Jatin.
- Surya MUST NOT modify frontend code unless explicitly authorised by the CTO.
- Charu Zaveri owns design deliverables, not application source code.
- Mohan Chawala owns test strategy and QA verification, not product implementation.
- The AI Code Reviewer reviews independently and does not silently fix reviewed code.
- Jatin coordinates and delegates; he does not invent requirements or override technical ownership.

## Workflow and approvals
- Work only on an issue assigned or explicitly delegated to the agent.
- Read the assigned issue, description, acceptance criteria, dependencies and comments before starting.
- If acceptance criteria are missing or ambiguous, ask Jatin to clarify with the Product Owner.
- Do not start out-of-scope work.
- Do not merge to `main`, deploy, or make irreversible production changes without explicit CTO approval.
- A task is not complete merely because code was written.
- Record meaningful progress, blockers, decisions, test evidence and handoffs in the work-tracking system agreed with Jatin.
- Do not mark work Done until required review, QA and acceptance gates are satisfied.

## Task completion report
Every agent must report:
1. Summary of work completed.
2. Files or deliverables changed.
3. Tests/checks performed and their results.
4. Outstanding issues, risks or limitations.
5. Required next owner or handoff.
6. Relevant branch, commit or pull request reference, when applicable.

## Safe collaboration
- Do not overwrite or discard another agent's work.
- Inspect the working tree before editing.
- Avoid unrelated formatting or refactoring.
- Coordinate shared-file changes before editing.
- Never claim a tool action, test, review or work-tracking update succeeded unless it actually did.
