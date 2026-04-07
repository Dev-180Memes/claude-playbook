# Architecture for [Project Name]

## Overview

This document outlines the Feature-Based (Vertical Slice) architecture for [Project Name]. Instead of grouping files by their technical role (all controllers together, all services together), files are grouped by the feature they belong to. Everything a feature needs — its controller, service, validation, tests, types — lives in one folder.

```text
HORIZONTAL (avoid)              VERTICAL SLICE (this pattern)
src/
├── controllers/                src/
│   ├── user.controller.js      ├── features/
│   ├── order.controller.js     │   ├── user/
│   └── product.controller.js  │   │   ├── user.controller.js
├── services/                  │   │   ├── user.service.js
│   ├── user.service.js        │   │   └── user.test.js
│   ├── order.service.js       │   ├── order/
│   └── product.service.js     │   │   ├── order.controller.js
└── models/                    │   │   ├── order.service.js
    ├── user.model.js           │   │   └── order.test.js
    ├── order.model.js          │   └── product/
    └── product.model.js        │       └── ...
```

The guiding principle is **high cohesion over horizontal layering**. A developer working on the `order` feature opens one folder and finds everything they need.

## When to Use

Use Feature-Based architecture when:

- The project has many distinct features that are owned by different developers or teams.
- Features change independently — the `billing` feature changing should require touching only the `billing` folder.
- You want to make it easy to delete or extract a feature (e.g., spin it out into a microservice).
- Horizontal layers (all services in one folder) have become so large that finding files is painful.

Avoid Feature-Based architecture when:

- The project is small with only 2–3 features — the structure adds navigation overhead without benefit.
- Features are tightly coupled and share so much logic that vertical slices would be largely empty or repetitive.

## Key Rules

- **Everything a feature needs lives in the feature's folder.** Controller, service, validation, DTOs, tests, types — all co-located.
- **Features must not import from each other's internals.** If `order` needs something from `user`, it must import from the `user` feature's public index file, not from a deep internal file like `user/user.service.js`.
- **Each feature folder must have an index file** that explicitly declares its public surface (what other features are allowed to import). Anything not exported from `index.js` is private to the feature.
- **Shared code lives in a `shared/` or `common/` folder, not in a feature.** Utilities, base error classes, and shared types that two or more features use belong in shared. If only one feature uses something, it stays in that feature.
- **Shared must not import from features.** Dependencies only flow from features → shared, never from shared → features. Circular dependencies between features are forbidden.
- **Tests live next to the code they test.** A `user.service.test.js` file sits in the `user/` folder, not in a top-level `tests/` directory.

## Data Flow

```text
HTTP Request
    ↓
Router (app-level)    → routes to the feature's Controller
    ↓
Feature Controller    → validates input, calls Feature Service
    ↓
Feature Service       → applies logic, calls DB or shared utilities
    ↓
Database
    ↓
Feature Service       → returns result
    ↓
Feature Controller    → maps to response DTO, sends HTTP response
    ↓
HTTP Response
```

Inter-feature data flow (when `order` needs user data):

```text
Order Service  →  user/index.js (public API)  →  User Service
```

`Order Service` never imports `user/user.service.js` directly.

## File Types

- **Feature Controller File:** HTTP entry/exit point for the feature. Lives inside the feature folder.
- **Feature Service File:** Business logic for the feature. Lives inside the feature folder.
- **Feature Repository File:** (Optional) DB access for the feature. Include only if the feature warrants the abstraction.
- **Feature DTO File:** Request and response shapes for the feature.
- **Feature Validation File:** Input validation specific to the feature.
- **Feature Errors File:** Custom error classes specific to the feature.
- **Feature Router File:** Route definitions for the feature. Registered in the app-level router.
- **Feature Test File:** Unit and integration tests co-located with the feature code.
- **Feature Index File:** The feature's public API. The only file other features are allowed to import from.
- **App-Level Router File:** Aggregates all feature routers and mounts them. Lives at the `src/` root, not inside any feature.
- **Shared Utility File:** Functions used by more than one feature. Lives in `shared/`.
- **Shared Error File:** Base error classes or generic errors (e.g., `NotFoundError`) used across features. Lives in `shared/`.
- **Shared Types / Constants File:** Type definitions or constants used across features. Lives in `shared/`.

## Folder Conventions

```text
src/
├── features/
│   └── [feature-name]/
│       ├── [feature].controller.js     ← HTTP layer
│       ├── [feature].service.js        ← business logic
│       ├── [feature].repository.js     ← (optional) DB layer
│       ├── [feature].router.js         ← route definitions
│       ├── [feature].dto.js            ← request/response shapes
│       ├── [feature].validation.js     ← input validation
│       ├── [feature].errors.js         ← feature-specific errors
│       ├── [feature].test.js           ← tests
│       └── index.js                    ← public surface
├── shared/
│   ├── utils.js
│   ├── errors.js
│   └── constants.js
└── app.js                              ← mounts all feature routers
```

## Example

Here is the architecture for a project with `user`, `order`, and `notification` features:

```text
src/
├── features/
│   ├── user/
│   │   ├── user.controller.js
│   │   ├── user.service.js
│   │   ├── user.router.js
│   │   ├── user.dto.js
│   │   ├── user.validation.js
│   │   ├── user.errors.js
│   │   ├── user.test.js
│   │   └── index.js              ← exports: UserService, userRouter
│   ├── order/
│   │   ├── order.controller.js
│   │   ├── order.service.js
│   │   ├── order.repository.js
│   │   ├── order.router.js
│   │   ├── order.dto.js
│   │   ├── order.validation.js
│   │   ├── order.errors.js
│   │   ├── order.test.js
│   │   └── index.js              ← exports: OrderService, orderRouter
│   └── notification/
│       ├── notification.service.js
│       ├── notification.errors.js
│       ├── notification.test.js
│       └── index.js              ← exports: NotificationService
├── shared/
│   ├── utils.js
│   ├── errors.js
│   └── db.js
└── app.js
```

### What each file does

- `user/index.js` — exports `UserService` and `userRouter`. Nothing else from the `user` folder is accessible to other features.
- `order/order.service.js` — when it needs to look up a user, it imports from `user/index.js`:

```js
import { UserService } from '../user/index.js'   // ✓ correct
// import { UserService } from '../user/user.service.js'  ✗ bypass of public API
```

- `app.js` — registers all feature routers:

```js
import { userRouter }         from './features/user/index.js'
import { orderRouter }        from './features/order/index.js'

app.use('/users',  userRouter)
app.use('/orders', orderRouter)
```

- `shared/errors.js` — defines `NotFoundError`, `ValidationError` etc. Features extend these but never define their own base classes.
- `notification/index.js` — exports only `NotificationService`. The notification feature has no router because it is triggered internally, not via HTTP.
