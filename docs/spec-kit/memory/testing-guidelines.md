# Testing Guidelines

> How we satisfy Principle 12 (Testability) and Principle 13 (Deterministic Ranking).

## Principles

- Every business rule must have automated tests.
- Tests verify **behavior**, not implementation details.
- The domain must be testable without a real database, frontend or external APIs.

## Mandatory test areas

- Prediction deadlines (prediction locking)
- Score calculation
- Ranking calculation
- Tie-breaking rules
- Authorization rules
- Admin actions
- Match result updates
- Audit creation
- Error scenarios

## Strategy per layer

| Layer        | Test type                          | Tool                  |
| ------------ | ---------------------------------- | --------------------- |
| Domain       | Unit (no I/O, deterministic)       | Jest                  |
| Use cases    | Unit with mocked dependencies      | Jest                  |
| API          | Integration against OpenAPI        | Jest + supertest      |
| UI           | Component / interaction            | React Testing Library |
| End-to-end   | Critical user flows                | Cypress               |

## Determinism rules

- Ranking calculation must be reproducible: same facts → same leaderboard.
- Inject the "clock" as a dependency to test deadlines without waiting real time.
