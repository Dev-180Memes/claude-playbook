# Architecture for [Project Name]

## Overview

This document outlines the Object-Oriented Programming (OOP) architecture for [Project Name]. The application is divided into self-contained modules, each encapsulating its own data and behaviour. Modules communicate through well-defined interfaces, keeping coupling low and cohesion high.

## When to Use

Use this architecture when:

- The domain maps naturally to objects with state and behaviour (e.g., users, orders, products).
- You want a modular structure where each feature area is independently testable and replaceable.
- The codebase is medium-to-large and needs clear ownership boundaries between feature teams.

Avoid this architecture for simple CRUD APIs with no meaningful domain logic — plain MVC will be less overhead.

## Key Rules

- Every feature must be divided into its own module. Each module has a single responsibility.
- Each module must have a service file as its core — this is where business logic lives.
- Controllers handle input/output only. They must not contain business logic.
- If a module needs database access, it must go through a repository file that abstracts the DB layer from the service.
- Modules must interact through interfaces, not concrete implementations. This makes them swappable and testable.
- Circular dependencies between modules are strictly forbidden. If two modules need to share behaviour, extract it into a shared module.
- All inter-module dependencies should be injected (dependency injection), not instantiated inside the consuming class.

## Data Flow

```text
HTTP Request
    ↓
Router          → routes to the correct Controller method
    ↓
Controller      → validates input (via Validation), calls Service
    ↓
Service         → executes business logic, calls Repository if needed
    ↓
Repository      → reads/writes to the database, returns domain objects
    ↓
Service         → maps result to DTO
    ↓
Controller      → returns response
    ↓
HTTP Response
```

## File Types

- **Service File:** Contains the business logic of the module. The single source of truth for what the module does.
- **Controller File:** Handles HTTP input and output. Delegates all processing to the Service. Must be kept thin.
- **Model/Entity File:** Defines the shape of the domain object and its database schema.
- **Repository File:** Abstracts all database interactions. The Service calls the Repository; it never queries the DB directly.
- **Interface File:** Defines the contract for interaction between modules. Depend on the interface, not the implementation.
- **DTO File:** Shapes data crossing module or layer boundaries. Keeps internal structures from leaking outward.
- **Validation File:** Validates incoming request data before it reaches the Service.
- **Error Handling File:** Defines module-specific custom error classes and centralised error handling.
- **Event/Message File:** Defines events or messages for inter-module communication in event-driven flows.
- **Utility File:** Shared helper functions with no module-specific dependencies.
- **Constants/Config File:** Module-scoped constants and configuration values.
- **Test File:** Unit and integration tests for the module's service, controller, and repository.
- **Index/Barrel File:** Re-exports the module's public API for cleaner imports elsewhere.

## Component Descriptions

- **Service:** The heart of the module. Contains all business rules and orchestrates calls to repositories and external dependencies. Other modules should depend on a service's interface, not its concrete class.
- **Controller:** The entry point for HTTP requests. Reads input, calls the service, and writes the response. Should contain no `if` statements driven by business rules.
- **Repository:** The only component allowed to speak to the database. Returns plain domain objects to the service, hiding query language details entirely.
- **Interface:** The contract a module exposes to the outside world. Swap implementations (e.g., swap a real email service for a mock) without touching consumers.

## Example

Here is how the architecture looks for a user management module:

```text
src/
└── user-module/
    ├── user.service.js
    ├── user.controller.js
    ├── user.model.js
    ├── user.repository.js
    ├── user.interface.js
    ├── user.dto.js
    ├── user.validation.js
    ├── user.errors.js
    ├── user.test.js
    └── index.js
```

- `user.service.js` — contains `createUser`, `getUserById`, `deactivateUser`, etc.
- `user.controller.js` — handles `POST /users`, reads `req.body`, calls `UserService.createUser`, returns response.
- `user.model.js` — defines the `User` entity and its database schema.
- `user.repository.js` — implements `findById`, `save`, `delete` against the database.
- `user.interface.js` — declares `IUserService` and `IUserRepository` so other modules depend on contracts.
- `user.dto.js` — defines `CreateUserDTO` and `UserResponseDTO`.
- `user.validation.js` — validates `CreateUserDTO` fields before the controller passes them to the service.
- `user.errors.js` — defines `UserNotFoundError`, `DuplicateEmailError`, etc.
- `user.test.js` — unit tests for the service (with a mocked repository) and integration tests for the repository.
- `index.js` — exports `UserService`, `IUserService`, and `UserController` as the module's public surface.
