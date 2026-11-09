# Bug Fixing Workflow

1. Mohan or another agent records the defect in the work-tracking system with reproduction steps and evidence.
2. Jatin triages severity, impact, ownership and priority.
3. The issue is assigned to the owner of the affected layer:
   - Frontend: Prashant.
   - Backend: Surya.
   - Design/usability: Charu provides design guidance; implementation goes to the relevant developer.
4. The developer reproduces the issue, identifies root cause and proposes a minimal fix.
5. Cross-boundary fixes are coordinated; ownership boundaries remain in force.
6. Developer adds or updates regression tests and submits changes for review.
7. AI Code Reviewer reviews the diff.
8. Mohan retests and runs targeted regression.
9. Jatin reports the outcome and escalates unresolved risk.
10. Product Owner accepts material risk or approves exceptional scope changes.

Never close a defect solely because a code change was made.
