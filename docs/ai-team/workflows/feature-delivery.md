# Feature Delivery Workflow

## 1. Intake and clarification
- Product Owner creates or approves the feature.
- Jatin checks scope, user value, dependencies and acceptance criteria.
- Ambiguity is returned to the Product Owner; agents must not invent business rules.

## 2. Breakdown and design
- Jatin creates appropriately scoped subtasks.
- Charu prepares user flows and design specifications where UI changes are involved.
- Design questions affecting product behaviour are escalated for approval.

## 3. API contract and implementation
- Surya owns backend implementation and API contracts.
- Prashant owns frontend implementation and API consumption.
- Frontend and backend may work in parallel only after the contract and dependencies are agreed.
- Prashant must not change backend files. Backend gaps go to Surya through Jatin.
- Both developers follow `mimir-skill`.

## 4. Code review
- Implementation is submitted as a branch/PR.
- AI Code Reviewer independently reviews the diff.
- CHANGES REQUESTED returns to the responsible developer.
- Re-review is required after material changes.
- No self-approval.

## 5. QA
- Mohan tests against acceptance criteria and relevant regression scenarios.
- FAIL creates or updates defects assigned to the responsible developer.
- Fixed defects require retest and appropriate regression.

## 6. Acceptance and closure
- Jatin consolidates implementation, review and QA evidence.
- Product Owner reviews the outcome and accepts the feature.
- Only then may the issue be marked Done.
- Merge and deployment require explicit CTO approval.

## Parallel work
Design, backend discovery and frontend scaffolding can proceed in parallel where safe. Shared files, API contracts and dependent work must be coordinated to avoid conflicting changes.
