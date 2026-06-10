# API Contracts

> Verifiable agreements between client and server. OpenAPI mandatory,
> all endpoints documented, errors normalized.

## Endpoint Contract Template

Every endpoint must specify:

- Purpose
- Method
- Path
- Authentication
- Authorization
- Request Params
- Request Body
- Success Response
- Error Responses
- Business Rules
- Side Effects
- Audit Events

## Example

```
POST /users

Purpose:
Create a new user.

Method:
POST

Path:
/users

Authentication:
Required.

Authorization:
Only admin users.

Request Body:
{
  "name": "string",
  "email": "string"
}

Success Response:
201 Created

Error Responses:
400 Validation error
401 Unauthenticated
403 Unauthorized
409 Email already exists

Business Rules:
- Email must be unique.
- Name cannot be empty.
```
