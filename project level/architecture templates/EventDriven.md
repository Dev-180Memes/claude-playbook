# Architecture for [Project Name]

## Overview

This document outlines the Event-Driven Architecture (EDA) for [Project Name]. Modules do not call each other directly. Instead, when something meaningful happens, a module emits an event. Any module that cares about that event subscribes to it and reacts. This eliminates hard dependencies between modules — the emitter never knows who is listening.

```text
[Order Module]  →  "order.placed"  →  [Event Bus]  →  [Inventory Module]
                                                    →  [Notification Module]
                                                    →  [Analytics Module]
```

## When to Use

Use Event-Driven Architecture when:

- Modules need to react to each other's actions but must remain independently deployable and testable.
- Side effects of an action are additive over time (new subscribers can be added without changing the emitter).
- You want to decouple write operations from downstream consequences (e.g., place order → update inventory, send email, log analytics all happen independently).
- The system is large enough that direct inter-module calls are creating tangled dependencies.

Avoid EDA when:

- The flow is simple and linear — direct service calls are easier to trace and debug.
- You need a synchronous response from a downstream action (EDA is fire-and-forget by default).
- The team is small and the overhead of an event bus and async flows is not justified.

## Key Rules

- **Emitters are ignorant of subscribers.** A module that emits an event must not import or know about the modules that handle it.
- **Events are immutable records.** An event describes something that already happened. It must never be mutated after being emitted. Name events in the past tense (e.g., `order.placed`, `user.registered`).
- **Each event must have a defined schema.** Consumers rely on the event shape. Breaking changes to an event schema are treated the same as a breaking API change.
- **Handlers must be idempotent.** Events may be delivered more than once (at-least-once delivery). Handlers must produce the same result whether they process an event once or multiple times.
- **Handlers must not emit events synchronously within another handler** unless the chain is explicit and documented. Accidental event chains are a debugging nightmare.
- **Failed events must be routed to a dead-letter queue (DLQ).** Never silently swallow a handler error.
- **Each handler has a single responsibility.** One handler, one reaction. Do not put multiple concerns in a single listener.

## Data Flow

```text
User Action / API Call
    ↓
Command Handler / Service
    ↓ emits
Event Bus            → delivers event to all registered subscribers
    ↓                                          ↓                      ↓
Inventory Handler               Notification Handler         Analytics Handler
    ↓                                          ↓
Updates stock                          Sends email
    ↓
(optionally emits "inventory.updated")
```

Each handler runs independently. A failure in `NotificationHandler` does not affect `InventoryHandler`.

## File Types

- **Event File:** Defines the schema and name of a single event. Acts as the contract between emitter and subscribers.
- **Event Bus File:** The central broker. Provides `emit(event)` and `subscribe(eventName, handler)`. May wrap an in-process emitter (e.g., EventEmitter) or an external broker (e.g., SQS, RabbitMQ, Kafka).
- **Publisher / Emitter File:** The module-level wrapper around the event bus that a service uses to emit events. Keeps services from depending on the bus directly.
- **Handler / Listener File:** Contains the logic that runs in response to a specific event. One handler per event per module.
- **Service File:** Contains business logic. Calls the publisher after a domain action succeeds. Does not call other services directly.
- **Controller File:** HTTP entry point. Delegates to the service. Has no knowledge of events.
- **Router File:** Maps HTTP routes to controllers.
- **DTO File:** Shapes for incoming HTTP data and outgoing responses. Separate from event payloads.
- **Dead-Letter Queue (DLQ) Handler File:** Handles events that failed processing. Logs, alerts, or retries as appropriate.
- **Test File:** Unit tests for handlers (using a mock event bus) and integration tests that verify the full emit → handle chain.
- **Index/Barrel File:** Re-exports the module's public surface (service, event names, handler registrations).

## Component Descriptions

- **Event Bus:** The nervous system of the application. All events flow through it. It must be the only channel for inter-module communication.
- **Publisher:** A thin wrapper a service uses to emit events. Decouples the service from the bus implementation so the bus can be swapped (in-memory vs Kafka) without changing service code.
- **Handler:** The unit of reaction. It receives one event type and performs one action. It should read like a plain description of the side effect: "when an order is placed, reserve inventory."
- **Service:** Executes business logic and emits an event on completion. It never calls another module's service — it only emits.

## Example

Here is how the architecture looks for an order flow (`order placed → inventory updated → notification sent`):

```text
src/
├── events/
│   ├── event-bus.js
│   ├── order.events.js
│   └── inventory.events.js
├── order-module/
│   ├── order.service.js
│   ├── order.publisher.js
│   ├── order.controller.js
│   ├── order.router.js
│   ├── order.dto.js
│   ├── order.errors.js
│   ├── order.test.js
│   └── index.js
├── inventory-module/
│   ├── inventory.service.js
│   ├── inventory.handler.js       ← subscribes to "order.placed"
│   ├── inventory.publisher.js
│   ├── inventory.errors.js
│   ├── inventory.test.js
│   └── index.js
└── notification-module/
    ├── notification.service.js
    ├── notification.handler.js    ← subscribes to "order.placed"
    ├── notification.errors.js
    ├── notification.test.js
    └── index.js
```

### What each file does

- `event-bus.js` — exports a singleton `EventBus` with `emit(event)` and `on(eventName, handler)`.
- `order.events.js` — exports `ORDER_PLACED = 'order.placed'` and the `OrderPlacedEvent` schema `{ orderId, userId, items, total }`.
- `order.service.js` — contains `placeOrder()`. After saving the order, calls `OrderPublisher.orderPlaced(order)`. Never imports `InventoryService` or `NotificationService`.
- `order.publisher.js` — wraps `EventBus.emit()` with typed helpers: `orderPlaced(order)` builds the event and emits it.
- `inventory.handler.js` — registers `EventBus.on('order.placed', handler)`. Calls `InventoryService.reserveStock(items)` in the handler body.
- `notification.handler.js` — registers `EventBus.on('order.placed', handler)`. Calls `NotificationService.sendOrderConfirmation(userId)`.
- Handler registrations are bootstrapped in each module's `index.js` so they're active when the application starts.
