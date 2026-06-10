# Backend Guidelines

> How we apply the constitution's laws in the service layer.

## General rules

- Layered architecture.
- DTOs mandatory.
- Input validation.
- Services with no direct dependency on Express / the HTTP framework.

## Service Contract

Every use case must declare:

- Input
- Output
- Preconditions
- Postconditions
- Domain Errors
- External Dependencies

### Example

```
Use Case: CreateUser

Preconditions:
- Email format is valid.
- Email does not already exist.

Postconditions:
- User is persisted.
- UserCreated event is emitted.
- Audit log is created.

Domain Errors:
- InvalidEmail
- EmailAlreadyExists
```
