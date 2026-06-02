# Project Constitution

## Architecture Principles

- Every feature begins as an isolated module before integration into the application.
- Frontend modules must expose public components, hooks, and services through explicit index files.
- Backend modules must follow layered architecture: routes → controllers → services → repositories.
- Modules must not import internal files from other modules directly.
- Shared logic must live in a dedicated `shared`, `common`, or `core` layer.
- All async operations must use consistent error handling with typed results or controlled exceptions.
- Database access must happen only through repositories—no direct queries inside controllers or services.
- Redis must be accessed only through a cache/service abstraction layer.
- API contracts between frontend and backend must be explicit and documented.

## Technology Constraints

- Frontend: React with TypeScript.
- UI Library: Ant Design for complex components such as forms, tables, modals, steps, tabs, and notifications.
- Styling: Tailwind CSS for layout, spacing, responsive design, and utility styling.
- Backend: Node.js with Express and TypeScript.
- Database: PostgreSQL.
- Cache/Queue: Redis.
- API format: REST JSON APIs by default.
- State management: Prefer React Query for server state; use Zustand, Redux Toolkit, or Context only when justified.
- Forms: Use Ant Design Form or React Hook Form, but do not mix both in the same feature.
- Validation: Use Zod, Yup, or Joi consistently across the project.
- Environment configuration must be handled through `.env` files and validated at startup.

## Frontend Standards

- All React components must be written in TypeScript using `.tsx`.
- Components must be small, focused, and reusable.
- Container/page components handle orchestration; presentational components handle UI only.
- API calls must be centralized in service files, never inside deeply nested components.
- Ant Design components should be customized using tokens, theme configuration, or Tailwind wrappers.
- Tailwind must not be used to override Ant Design internals in a fragile way.
- Avoid inline styles unless required for dynamic values.
- Use custom hooks for reusable logic.
- Use loading, empty, error, and success states in every data-driven screen.
- Do not store server state manually if React Query or equivalent is being used.

## Backend Standards

- Express routes must only define endpoints and middleware composition.
- Controllers must only handle request/response mapping.
- Services must contain business logic.
- Repositories must contain database access logic.
- Middleware must be used for authentication, authorization, validation, logging, and error handling.
- All endpoints must validate body, params, and query data before reaching the controller.
- API responses must follow a consistent structure:

```ts
{
  success: boolean;
  data?: unknown;
  error?: {
    code: string;
    message: string;
    details?: unknown;
  };
}