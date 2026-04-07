# Architecture for [Project Name]

## Overview

This document outlines the Clean Architecture (also known as Hexagonal Architecture or Ports & Adapters) for [Project Name]. Business logic lives at the centre and knows nothing about the outside world. Everything external — databases, HTTP, third-party services — lives on the outer ring and plugs in through defined ports (interfaces). This means infrastructure can be swapped without touching a single line of business logic.

The dependency rule is absolute: **outer layers depend on inner layers. Inner layers never depend on outer layers.**

```text
┌─────────────────────────────────────────┐
│           Infrastructure                │  ← DB, HTTP, email, queues
│   ┌─────────────────────────────────┐   │
│   │         Application             │   │  ← Use cases, ports (interfaces)
│   │   ┌─────────────────────────┐   │   │
│   │   │         Domain          │   │   │  ← Entities, business rules
│   │   └─────────────────────────┘   │   │
│   └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

## When to Use

Use Clean Architecture when:

- The domain is complex and the business rules need to be protected from infrastructure concerns.
- You anticipate swapping infrastructure (e.g., switching from PostgreSQL to MongoDB, or REST to GraphQL) without rewriting business logic.
- You need the domain and application layers to be fully unit-testable without spinning up a database or HTTP server.
- Multiple delivery mechanisms are needed (e.g., HTTP API + CLI + message consumer) all sharing the same use cases.

Avoid Clean Architecture for simple CRUD applications — the added structure is overhead without a complex domain to justify it.

## Key Rules

- **The Dependency Rule:** Source code dependencies must point inward only. Domain knows nothing about Application. Application knows nothing about Infrastructure.
- **Domain is pure:** Entities and domain services must have zero imports from any framework, ORM, or external library. They must be plain objects/classes.
- **Ports are interfaces defined inside the application layer.** They describe what the application needs (e.g., `IUserRepository`, `IEmailService`). They make no assumptions about how those needs are fulfilled.
- **Adapters are implementations defined in the infrastructure layer.** They implement the ports. The application layer never imports an adapter directly — dependency injection wires them together at startup.
- **Use cases are the application's API.** Each use case represents one thing the system can do (e.g., `CreateUserUseCase`, `SendPasswordResetUseCase`). They orchestrate domain objects and call ports.
- **No business logic in controllers or adapters.** Controllers parse input and hand it to a use case. Adapters translate between external formats and domain objects.
- **DTOs cross layer boundaries.** Domain objects must never be serialised directly into HTTP responses or database rows.

## Layers

### Domain Layer (innermost)

The core of the application. Contains the business rules that would exist regardless of any technology choices.

- **Entities:** Business objects with identity and lifecycle (e.g., `User`, `Order`). Contain domain logic as methods.
- **Value Objects:** Immutable descriptors with no identity (e.g., `Email`, `Money`, `Address`). Equality is by value.
- **Domain Services:** Stateless operations that involve multiple entities or don't belong to a single entity (e.g., `PricingService`).
- **Domain Events:** Signals that something meaningful happened in the domain (e.g., `UserRegisteredEvent`).
- **Domain Errors:** Business rule violations expressed as typed errors (e.g., `InsufficientFundsError`).

### Application Layer

Orchestrates the domain to fulfil the system's use cases. Knows about the domain but not about infrastructure.

- **Use Cases:** One class per operation. Receives a request DTO, calls domain objects and ports, returns a response DTO.
- **Ports (Interfaces):** Interfaces that describe external dependencies the use cases need. Split into:
  - *Driven ports (output):* things the application calls outward, e.g., `IUserRepository`, `IEmailService`.
  - *Driving ports (input):* optional — interfaces that describe how the application can be invoked, e.g., `ICreateUserUseCase`.
- **Application DTOs:** Input and output shapes for use cases. Separate from HTTP request/response shapes.

### Infrastructure Layer (outermost)

Implements the ports. Contains all framework, library, and third-party code.

- **Repository Adapters:** Implement `IUserRepository` using an ORM, SQL client, or document store.
- **Service Adapters:** Implement `IEmailService` using SendGrid, SES, etc.
- **Persistence Schemas:** ORM models or migration files. These are infrastructure concerns, not domain objects.
- **Message Adapters:** Implement queue producers/consumers (e.g., SQS, RabbitMQ).

### Presentation Layer

The delivery mechanism. Translates external input into use case calls and use case results into external output.

- **Controllers:** Parse HTTP requests, call a use case, map the result to an HTTP response. No business logic.
- **Routers:** Define HTTP routes and bind them to controllers.
- **Request Validators:** Validate the raw HTTP payload before it reaches a controller.
- **Presenters / Serialisers:** Map application DTOs to HTTP response shapes.

## Data Flow

```text
HTTP Request
    ↓
Router              → matches route, invokes Controller
    ↓
Controller          → validates input, builds request DTO, calls Use Case
    ↓
Use Case            → orchestrates Domain entities, calls Ports (interfaces)
    ↓
Repository Port     → implemented by Repository Adapter → reads/writes DB
    ↓
Use Case            → returns response DTO
    ↓
Controller          → maps DTO to HTTP response via Presenter
    ↓
HTTP Response
```

At no point does the Use Case import from the Repository Adapter. The adapter is injected at startup via a DI container or manual wiring.

## File Types

- **Entity File:** A domain object with identity. Contains business methods, not persistence logic.
- **Value Object File:** An immutable domain concept. Validates itself on construction.
- **Domain Service File:** Stateless domain logic that spans multiple entities.
- **Domain Event File:** A record of something that happened in the domain. Used for side-effect triggers.
- **Domain Errors File:** Typed errors representing broken business rules.
- **Use Case File:** One class, one operation. Orchestrates domain and ports to fulfil a single user intent.
- **Port File (Interface):** The interface that a use case depends on. Defined in the application layer.
- **Adapter File:** The concrete implementation of a port. Lives in infrastructure.
- **Persistence Schema File:** ORM model or migration. Infrastructure-only — never imported by domain or application layers.
- **Controller File:** HTTP entry point. Thin — delegates to a use case immediately.
- **Router File:** Maps HTTP routes to controllers.
- **Request Validator File:** Validates raw HTTP input before the controller processes it.
- **DTO File:** Data shapes that cross layer boundaries. Input DTOs enter use cases; output DTOs leave them.
- **Test File:** Domain and application layers should have pure unit tests (no DB, no HTTP). Infrastructure adapters get integration tests.
- **DI / Composition Root File:** The one place where ports are wired to adapters. This is the only file that imports from both application and infrastructure layers simultaneously.

## Example

Here is how the architecture looks for a user management feature:

```text
src/
├── domain/
│   └── user/
│       ├── user.entity.js
│       ├── email.value-object.js
│       ├── user.errors.js
│       └── user.domain-event.js
├── application/
│   └── user/
│       ├── use-cases/
│       │   ├── create-user.use-case.js
│       │   └── get-user.use-case.js
│       ├── ports/
│       │   ├── user-repository.port.js
│       │   └── email-service.port.js
│       └── dtos/
│           ├── create-user.dto.js
│           └── user-response.dto.js
├── infrastructure/
│   └── user/
│       ├── persistence/
│       │   ├── user.typeorm-repository.js
│       │   └── user.schema.js
│       └── services/
│           └── sendgrid-email.service.js
├── presentation/
│   └── http/
│       ├── user.controller.js
│       ├── user.router.js
│       └── user.request-validator.js
└── composition-root.js
```

### What each file does

- `user.entity.js` — defines the `User` class with methods like `activate()`, `changeEmail(newEmail)`. No ORM imports.
- `email.value-object.js` — `Email` class that validates format on construction and compares by value.
- `user.errors.js` — `UserNotFoundError`, `DuplicateEmailError` as typed domain errors.
- `user.domain-event.js` — `UserRegisteredEvent` emitted after a user is created successfully.
- `create-user.use-case.js` — receives `CreateUserDTO`, calls `IUserRepository.save()` and `IEmailService.sendWelcome()`.
- `user-repository.port.js` — `IUserRepository` interface with `findById`, `findByEmail`, `save`, `delete`.
- `email-service.port.js` — `IEmailService` interface with `sendWelcome`, `sendPasswordReset`.
- `user.typeorm-repository.js` — implements `IUserRepository` using TypeORM. The only file that imports TypeORM.
- `sendgrid-email.service.js` — implements `IEmailService` using the SendGrid SDK.
- `user.controller.js` — calls `CreateUserUseCase` with validated input, returns HTTP 201 with response DTO.
- `composition-root.js` — wires `UserTypeOrmRepository` → `IUserRepository`, `SendGridEmailService` → `IEmailService`, injects both into `CreateUserUseCase`.
