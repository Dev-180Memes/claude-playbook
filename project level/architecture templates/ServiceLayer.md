# Architecture for [Project Name]

## Overview

This document outlines the Service Layer Pattern for [Project Name]. It is a deliberately flat, lightweight structure: HTTP controllers talk to services, services talk to the database, nothing else. There are no repositories abstracting the DB layer, no domain entities, no use case objects. The service is the single place where logic lives.

```text
HTTP Request  →  Controller  →  Service  →  Database (ORM / query builder)
HTTP Response ←  Controller  ←  Service  ←  Database result
```

This pattern trades abstraction depth for simplicity. It is the right choice when the domain is thin and the value of deeper layering has not been earned yet.

## When to Use

Use the Service Layer Pattern when:

- Building a microservice or simple REST API where business logic is mostly CRUD operations with light validation.
- The team is small or the project is early-stage and premature abstraction would slow delivery.
- The DB layer is thin and the likelihood of switching databases is low.
- You need to move fast and can refactor to a richer architecture later if complexity grows.

Avoid this pattern when:

- Business logic is complex enough that services are growing beyond 200–300 lines — a sign that domain objects or repositories would pay off.
- Multiple delivery mechanisms (HTTP + CLI + queue consumers) need to share logic — the use-case or hexagonal pattern is a better fit.
- Services are importing other services heavily, creating tangled dependency chains.

## Key Rules

- **No business logic in controllers.** Controllers parse input, call a service method, and return the result. They contain no `if` statements driven by business rules.
- **Services own all logic.** Validation, orchestration, DB reads and writes, and error handling all live in the service.
- **Services talk to the DB directly** via the ORM or query builder. There is no repository layer. If a service needs data, it queries for it.
- **Services must not call other services unless the dependency is clearly one-directional.** Circular service dependencies (A calls B, B calls A) are forbidden. If two services need shared logic, extract it into a shared utility or helper module, not into a third service.
- **One service per module.** The service file is the module. Group all database interaction, validation, and business logic for a feature in its service.
- **Keep services focused.** If a service file grows past 300 lines, it is a signal that the module is doing too much. Split by responsibility, not by layer.

## Data Flow

```text
HTTP Request
    ↓
Router          → matches route, invokes Controller method
    ↓
Controller      → reads request body/params, calls Service
    ↓
Service         → validates input, applies logic, queries/writes DB via ORM
    ↓
Database        → returns rows or acknowledgement
    ↓
Service         → maps DB result to response shape
    ↓
Controller      → sends HTTP response
    ↓
HTTP Response
```

## File Types

- **Service File:** The core of the module. Contains validation, business logic, and direct DB access. Every feature's logic lives here.
- **Controller File:** HTTP entry and exit point. Reads request data, calls the service, sets the HTTP response. Must be kept thin.
- **Router File:** Maps HTTP methods and paths to controller functions.
- **DTO File:** Shapes for request bodies and response payloads. Keeps the controller's surface area explicit.
- **Validation File:** Validates incoming request data. Called by the controller before the service.
- **Error Handling File:** Defines custom error classes (e.g., `NotFoundError`, `ConflictError`) and centralised error middleware.
- **Utility File:** Shared helpers used across services (e.g., `paginate`, `hashPassword`, `formatDate`). No feature-specific logic.
- **Constants/Config File:** Environment-driven configuration and application-wide constants.
- **Test File:** Integration tests that call the service directly (or via HTTP) against a real or in-memory database. Unit test individual service methods where logic is complex.
- **Index/Barrel File:** Re-exports the module's controller and service for cleaner registration in the application root.

## Example

Here is how the architecture looks for a user management module:

```text
src/
├── user/
│   ├── user.service.js
│   ├── user.controller.js
│   ├── user.router.js
│   ├── user.dto.js
│   ├── user.validation.js
│   ├── user.errors.js
│   ├── user.test.js
│   └── index.js
├── shared/
│   ├── utils.js
│   └── errors.js
└── app.js
```

### What each file does

- `user.service.js` — contains `createUser`, `getUserById`, `updateUser`, `deleteUser`. Each method validates inputs, runs any logic (e.g., checks for duplicate email), and calls the ORM directly:

```js
async createUser({ email, password }) {
  const existing = await db('users').where({ email }).first()
  if (existing) throw new DuplicateEmailError(email)
  const hashed = await hashPassword(password)
  const [id] = await db('users').insert({ email, password: hashed })
  return this.getUserById(id)
}
```

- `user.controller.js` — handles `POST /users`, reads `req.body`, calls `UserService.createUser`, returns `201` with the created user. Contains no logic beyond that.
- `user.router.js` — maps `POST /users` → `UserController.create`, `GET /users/:id` → `UserController.getById`, etc.
- `user.dto.js` — defines `CreateUserRequestDTO` (`{ email, password }`) and `UserResponseDTO` (`{ id, email, createdAt }`).
- `user.validation.js` — validates `email` is a valid address, `password` meets length requirements, before the controller hands off to the service.
- `user.errors.js` — defines `DuplicateEmailError`, `UserNotFoundError`.
- `user.test.js` — integration tests for `UserService` against a test database, and HTTP-level tests for the router.
- `index.js` — exports `userRouter` for registration in `app.js`.
