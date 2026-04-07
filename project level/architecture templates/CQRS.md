# Architecture for [Project Name]

## Overview

This document outlines the Command Query Responsibility Segregation (CQRS) architecture for [Project Name]. The core idea is simple: **commands change state, queries read state — and the two are handled by completely separate models.**

On the write side, a command captures an intent (`CreateOrderCommand`), is validated, and mutates the domain. On the read side, a query goes directly to a read-optimised store or view and returns a flat DTO shaped for the UI — no domain objects, no business rules, just data.

```text
         WRITE SIDE                         READ SIDE
┌─────────────────────────┐      ┌─────────────────────────┐
│  Command → Handler      │      │  Query → Handler        │
│  Domain logic + rules   │      │  No domain, no logic    │
│  Writes to write store  │      │  Reads from read store  │
└────────────┬────────────┘      └──────────────┬──────────┘
             │ sync or async                     │
             ↓                                   ↓
       Write DB / Store              Read DB / View / Cache
             │
             └─── (sync or event-driven) ──────→ Read DB / View
```

The read store is a projection of the write store. In simple setups they can be the same database with separate query paths. In complex setups, the write store feeds events that build denormalised read views.

## When to Use

Use CQRS when:

- Read patterns differ significantly from write patterns (e.g., a dashboard needs aggregated, denormalised data but writes operate on normalised domain objects).
- The system has high read volume that needs to scale independently of writes.
- You are using Event Sourcing (CQRS and Event Sourcing are complementary but independent patterns).
- Complex reporting or analytics queries are polluting domain models.

Avoid CQRS when:

- The application is simple CRUD — CQRS adds two separate models for no gain.
- Read and write shapes are nearly identical — there is nothing to segregate.
- The team is small and the added complexity of maintaining two models is not justified.

## Key Rules

- **Commands express intent.** Name them in the imperative: `CreateOrderCommand`, `CancelSubscriptionCommand`. A command is a request to change state — it can be rejected.
- **Queries are side-effect free.** A query must never mutate state. If it does, it is a command.
- **Command handlers contain all write-side business logic.** They validate the command, apply domain rules, and persist the result. They must not return domain data — at most they return an ID or an acknowledgement.
- **Query handlers contain no business logic.** They read from the read store and return a DTO. No validation beyond the query's own shape.
- **The read model is expendable.** Because it is derived from the write side, it can always be rebuilt. Design read models for query performance, not for correctness.
- **Command and query models must not share domain objects.** The read side's DTO is not the write side's entity in disguise.
- **Keep the write side and read side synchronisation explicit.** Whether it is synchronous (same DB, different queries) or asynchronous (events updating a read store), document and test that boundary.

## Data Flow

### Write Path

```text
HTTP POST / PUT / DELETE
    ↓
Controller          → builds Command object from request body
    ↓
Command Bus         → routes Command to the correct Command Handler
    ↓
Command Handler     → validates, runs domain logic, writes to write store
    ↓
Write Store         → persists the state change
    ↓ (optional, for async read sync)
Domain Event        → emitted, picked up by Read Model Projector
    ↓
Read Store          → updated projection
```

### Read Path

```text
HTTP GET
    ↓
Controller          → builds Query object from request params
    ↓
Query Bus           → routes Query to the correct Query Handler
    ↓
Query Handler       → runs optimised read against read store
    ↓
Read Store          → returns flat, denormalised rows
    ↓
Controller          → returns DTO directly (no further mapping needed)
    ↓
HTTP Response
```

## File Types

- **Command File:** A plain data object that captures an intent to change state (e.g., `CreateOrderCommand`). No logic.
- **Command Handler File:** Validates the command, runs domain/business logic, writes to the write store. One handler per command.
- **Command Bus File:** Routes a command to its handler. May be a simple map or a framework-provided dispatcher.
- **Query File:** A plain data object that describes what data is needed and any filters/pagination (e.g., `GetOrdersByUserQuery`).
- **Query Handler File:** Reads from the read store and returns a DTO. No domain logic. One handler per query.
- **Query Bus File:** Routes a query to its handler.
- **Read Model / DTO File:** The flat, query-optimised shape returned to the caller. Designed for the consumer, not for the domain.
- **Write Model / Entity File:** The domain object used on the write side. Contains business rules and invariants.
- **Projector File:** Listens for domain events from the write side and updates the read store. Required only when the read store is separate from the write store.
- **Repository File:** Write-side persistence. Used by command handlers. Read-side query handlers may use direct DB queries instead of a repository.
- **Controller File:** Receives HTTP requests, builds the appropriate Command or Query, dispatches it, and returns the result.
- **Router File:** Maps HTTP routes to controllers.
- **Validation File:** Validates command/query input before it reaches the handler.
- **Test File:** Command handler tests verify business rule enforcement. Query handler tests verify the correct data shape is returned. Keep them separate.

## Example

Here is the architecture applied to an order management system:

```text
src/
├── write/
│   └── order/
│       ├── commands/
│       │   ├── create-order.command.js
│       │   ├── create-order.handler.js
│       │   ├── cancel-order.command.js
│       │   └── cancel-order.handler.js
│       ├── domain/
│       │   ├── order.entity.js
│       │   └── order.errors.js
│       └── order.write-repository.js
├── read/
│   └── order/
│       ├── queries/
│       │   ├── get-order.query.js
│       │   ├── get-order.handler.js
│       │   ├── list-orders-by-user.query.js
│       │   └── list-orders-by-user.handler.js
│       ├── dtos/
│       │   ├── order-detail.dto.js
│       │   └── order-summary.dto.js
│       └── order.projector.js
├── presentation/
│   └── order/
│       ├── order.controller.js
│       └── order.router.js
└── shared/
    ├── command-bus.js
    └── query-bus.js
```

### What each file does

- `create-order.command.js` — plain object: `{ userId, items, shippingAddress }`. No methods.
- `create-order.handler.js` — validates stock availability, creates an `Order` entity, calls `OrderWriteRepository.save()`, optionally emits `OrderPlacedEvent`.
- `order.entity.js` — `Order` class with `place()`, `cancel()`, `addItem()` methods and business rule enforcement.
- `get-order.query.js` — plain object: `{ orderId, requestingUserId }`.
- `get-order.handler.js` — runs a direct SQL/ORM query: `SELECT o.*, u.name FROM orders o JOIN users u ...`. Returns `OrderDetailDTO`. No `Order` entity involved.
- `order-detail.dto.js` — flat shape: `{ id, status, customerName, items: [...], total, placedAt }`. Shaped for the order detail page.
- `order.projector.js` — subscribes to `OrderPlacedEvent` and `OrderCancelledEvent`. Updates a denormalised `order_read_view` table so query handlers never join across write tables.
- `order.controller.js` — `POST /orders` → builds `CreateOrderCommand`, dispatches via `CommandBus`. `GET /orders/:id` → builds `GetOrderQuery`, dispatches via `QueryBus`, returns result directly.
