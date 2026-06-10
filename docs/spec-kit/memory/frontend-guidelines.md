# Frontend Guidelines

> How we apply the constitution's laws in the UI layer.

## General rules

- Functional components only.
- TypeScript mandatory.
- Typed props.
- No business logic inside components.
- Forms with React Hook Form.

## Component Creation

Every component must have a clear responsibility.

A component must be classified as:

- Page
- Layout
- Feature Component
- Shared Component
- Form Component
- UI Primitive

## Component Contract

Every reusable component must declare:

- Props
- Possible states
- Emitted events
- Error cases
- Loading states
- Empty states

### Example

```
Component: UserCard

Props:
- user: UserSummary
- onSelect?: (userId: string) => void
- variant?: "compact" | "detailed"

States:
- default
- loading
- empty
- error
```
