# Security Guidelines

> How we satisfy Principle 10 (Security by Design) and Principle 11 (Privacy).

## Input and output

- Every external input must be validated (type, format, range, authorization).
- Every sensitive output must be filtered: never expose data the consumer doesn't need.

## Sensitive operations

Every sensitive operation must:

- Authenticate the user.
- Authorize the specific action.
- Record the event (audit log).
- Avoid exposing sensitive data.

## Cross-cutting controls

- Authentication
- Authorization
- Input validation
- Output filtering
- Rate limiting for sensitive operations (login, prediction submit, recalculations)
- Protection against common web vulnerabilities (OWASP Top 10)

## Secrets

- Secrets are **never** stored in source code.
- They are managed via environment variables or a secrets manager.

## Privacy and data minimization

- Collect only the data needed to operate the pool.
- Leaderboards expose the minimum necessary public identity.
- Personal data is not exposed to other participants unless the product requires it.
