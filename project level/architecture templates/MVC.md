# Architecture for [Project Name]

## Overview

This document outlines the Model-View-Controller (MVC) architecture for [Project Name]. The application is divided into three layers — Model, View, and Controller — each with a distinct responsibility. This separation makes the codebase easier to test, maintain, and extend over time.

## When to Use

Use MVC when:

- Building a web application with a clear separation between data, presentation, and request handling.
- The team is familiar with the pattern and wants a widely understood convention.
- The domain logic is not overly complex and does not require deeper layering (e.g., use cases, ports/adapters).

Avoid MVC when the business logic is so rich that it bleeds into controllers or models — consider a layered or hexagonal architecture instead.

## Key Rules

- The application must be divided into three components: Model, View, and Controller. Do not mix their responsibilities.
- The Model owns data and business logic. It must never know about the View or Controller.
- The View owns presentation. It must never contain business logic or call the database directly.
- The Controller is the coordinator. It receives input, delegates to the Model, and passes results to the View. Keep it thin — logic belongs in the Model.
- Never let the View and Model communicate directly. All data flow must pass through the Controller.
- DTOs should be used to transfer data between layers so internal model structures are never leaked to the View.

## Data Flow

```text
HTTP Request
    ↓
Router        → routes to the correct Controller action
    ↓
Controller    → validates input (via Validation), calls Model
    ↓
Model         → executes business logic, reads/writes DB
    ↓
Controller    → maps result to DTO
    ↓
View          → renders response (HTML, JSON, etc.)
    ↓
HTTP Response
```

## File Types

- **Model File:** Defines data structure and business logic. Interacts with the database directly or via a query builder.
- **View File:** Handles presentation. Renders the final output (template, JSON serializer, etc.) from data passed by the Controller.
- **Controller File:** Handles incoming requests, coordinates the Model and View, and returns a response. Must be kept thin.
- **Router File:** Maps HTTP routes to Controller actions.
- **DTO File:** Shapes data passed between layers. Prevents internal model details from leaking into the View or external API.
- **Validation File:** Validates incoming request data before it reaches the Controller logic.
- **Error Handling File:** Defines custom error classes and centralised error handling middleware.
- **Utility File:** Shared helper functions with no layer-specific dependencies.
- **Constants/Config File:** Application-wide constants and environment-driven configuration.
- **Test File:** Unit and integration tests per component.
- **Index/Barrel File:** Re-exports components for cleaner imports across the application.

## Example

Here is how the architecture looks for a user management module:

```text
user-management/
├── user.model.js
├── user.view.js
├── user.controller.js
├── user.router.js
├── user.dto.js
├── user.validation.js
├── user.errors.js
├── user.util.js
├── user.test.js
└── index.js
```

- `user.model.js` — defines the User schema and any business methods (e.g., `hashPassword`, `isActive`).
- `user.view.js` — serialises a User model into the shape returned to the client.
- `user.controller.js` — handles `POST /users`, calls `user.model.js` to create a user, then delegates to the view.
- `user.router.js` — maps `POST /users` → `UserController.create`.
- `user.dto.js` — defines `CreateUserDTO` and `UserResponseDTO`.
- `user.validation.js` — validates the request body before the controller processes it.
- `user.errors.js` — defines `UserNotFoundError`, `DuplicateEmailError`, etc.
- `user.test.js` — unit tests for model methods and controller actions.
