# Constitution — World Cup Prediction Pool App

> These are the project's **non-negotiable laws**. Specs, plans, contracts and tasks
> must align with this document. Concrete technologies (React, Node.js, PostgreSQL,
> Redis…) do **not** live here: they live in `tech-stack.md` and `adr/`.

## 1. Contract-First Development

Every public feature must be defined by a verifiable contract before implementation.

Contracts may include:

- API contracts
- Data schemas
- Component contracts
- Business rules
- Acceptance criteria
- Error cases

No endpoint, prediction rule, scoring rule, ranking calculation, payment rule, or
user-facing workflow may be implemented without a prior contract.

## 2. Traceability

Every implementation must be traceable through the following chain:

`Requirement → Specification → Contract → Task → Test → Code`

A task without a related specification is invalid.

A test without a related business rule is incomplete.

## 3. Domain Integrity

The football prediction domain must remain independent from frameworks, UI libraries,
databases, and external services.

Core domain logic includes:

- Tournament configuration
- Match schedules
- Predictions
- Score calculation
- Ranking calculation
- Tie-breaking rules
- Group or league rules
- User participation rules

These rules must be testable without requiring a real database, frontend, payment
provider, or external football API.

## 4. Prediction Immutability

Predictions must become immutable once their configured deadline has passed.

The system must never allow users, admins, background jobs, or API clients to modify a
locked prediction.

Any exceptional correction must be handled through an auditable administrative process.

## 5. Transparent Scoring

All scoring rules must be explicit, documented, and reproducible.

A user must be able to understand why they received a specific score for a match.

The system must support explaining:

- Match result
- User prediction
- Applied scoring rule
- Earned points
- Ranking impact

No hidden scoring logic is allowed.

## 6. Fairness and Anti-Fraud

The system must protect competition fairness.

The system must prevent:

- Late predictions
- Prediction tampering
- Duplicate participation when forbidden
- Unauthorized admin changes
- Ranking manipulation
- Silent score recalculation

Every sensitive action must be authenticated, authorized, and audited.

## 7. Auditability

All critical actions must produce audit records.

Critical actions include:

- User registration
- Pool creation
- Pool invitation
- Prediction creation
- Prediction update before deadline
- Prediction locking
- Match result update
- Score recalculation
- Ranking recalculation
- Admin correction
- Payment or prize-related action, if applicable

Audit records must include:

- Actor
- Action
- Timestamp
- Previous state when applicable
- New state when applicable
- Reason when applicable

## 8. Explicit Time Handling

All time-sensitive rules must be based on a single authoritative time standard.

Prediction deadlines, match start times, result publication, and locking rules must use
explicit time zones.

The system must avoid ambiguous local-time behavior.

## 9. Error Handling by Contract

All expected errors must be declared in the corresponding contract.

Every endpoint or use case must define:

- Validation errors
- Authentication errors
- Authorization errors
- Business rule errors
- Conflict errors
- Unexpected errors

Business rule errors must be meaningful to users and developers.

## 10. Security by Design

All external input must be validated.

Sensitive data must never be exposed unnecessarily.

The system must enforce:

- Authentication
- Authorization
- Input validation
- Output filtering
- Rate limiting for sensitive operations
- Protection against common web vulnerabilities

Secrets must never be stored in source code.

## 11. Privacy and Data Minimization

The system must collect only the user data required to operate the prediction pool.

Personal data must be protected and should not be exposed to other participants unless
explicitly required by the product rules.

Leaderboards should expose only the minimum necessary public identity.

## 12. Testability

Every business rule must have automated tests.

Mandatory test areas include:

- Prediction deadlines
- Score calculation
- Ranking calculation
- Tie-breaking rules
- Authorization rules
- Admin actions
- Match result updates
- Audit creation
- Error scenarios

Tests must verify behavior, not implementation details.

## 13. Deterministic Ranking

Ranking calculation must be deterministic.

Given the same users, predictions, match results, and scoring rules, the system must
always produce the same leaderboard.

Tie-breaking rules must be explicit before the competition starts.

## 14. Administrative Accountability

Admin features must be limited, intentional, and auditable.

Admins may configure competitions, matches, rules, results, and corrections only through
authorized workflows.

No admin action may silently alter user points, predictions, or rankings.

## 15. Separation of Concerns

User interface, application logic, domain logic, infrastructure, and external
integrations must remain separated.

The system must avoid placing business rules inside:

- UI components
- Controllers
- Database queries
- External service adapters
- Background jobs without domain delegation

## 16. Recoverability

The system must be able to recover from operational mistakes.

Critical data must support backup, restoration, and recalculation when applicable.

Scores and rankings should be recalculable from source facts:

- Users
- Pools
- Matches
- Predictions
- Results
- Scoring rules

## 17. Observability

Critical operations must be observable.

The system must provide logs, metrics, or traces for:

- Prediction submissions
- Failed submissions
- Result updates
- Score recalculations
- Ranking recalculations
- Authentication failures
- Authorization failures
- Background jobs

## 18. Specification Governance

When a requested feature conflicts with this constitution, the constitution prevails.

To change a constitutional rule, the team must explicitly document:

- Reason for change
- Impacted specifications
- Migration strategy
- Updated contracts
- Updated tests

Silent constitutional changes are forbidden.
