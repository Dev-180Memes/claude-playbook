# Architecture for [Project Name]

## Overview

This document outlines the Repository Pattern for [Project Name]. A repository sits between your service (business logic) and your database. The service calls the repository using plain method names (`findById`, `save`, `delete`). The repository handles all query construction, ORM calls, and result mapping — invisibly. The service never knows whether the data is coming from PostgreSQL, MongoDB, an in-memory store, or a remote API.

```text
Service  →  IUserRepository (interface)  ←  UserPostgresRepository (concrete)
                                         ←  UserMongoRepository (concrete)
                                         ←  UserInMemoryRepository (for tests)
```

## When to Use

Use the Repository Pattern when:

- You want to keep business logic free of database-specific code (SQL, ORM decorators, query builders).
- You need to be able to unit-test services without hitting a real database.
- There is a realistic chance of changing the database or ORM in future.
- Multiple data sources need to be presented to the service through a uniform interface.

Avoid the Repository Pattern when:

- The application is a simple CRUD API with no meaningful business logic — the abstraction adds layers without benefit.
- You are using an Active Record ORM (e.g., Rails, Eloquent) where the model is already the repository by convention.

## Key Rules

- **The interface is defined in the application/service layer.** The service depends on the interface, not on any concrete repository class. The concrete class lives in the infrastructure/data layer.
- **Services never import the concrete repository.** The concrete implementation is injected — via a constructor, DI container, or factory — so the service only ever sees the interface.
- **Repositories return domain objects, not raw DB rows.** The repository is responsible for mapping a database record into the shape the service expects. ORM models and query results must never leak into service code.
- **One repository per aggregate root.** If `Order` contains `OrderLine` items, there is an `OrderRepository` — not a separate `OrderLineRepository`.
- **Repositories do not contain business logic.** They only translate between persistence and domain objects. A `findActiveUsers` method is acceptable; a `chargeUser` method is not.
- **Write a minimal in-memory implementation of the interface.** This is used in unit tests. It must behave identically to the real implementation for the cases the service cares about.

## File Types

- **Repository Interface File:** Defines the contract — the method signatures the service will call. Framework-free. Lives alongside the service.
- **Concrete Repository File:** Implements the interface using a specific database technology (PostgreSQL, MongoDB, Redis, etc.). Lives in the data/infrastructure layer.
- **In-Memory Repository File:** An in-process implementation of the interface backed by a plain array or map. Used exclusively in tests.
- **Persistence Schema / ORM Model File:** The ORM entity or migration. It is an infrastructure concern. The interface never references it.
- **Mapper File:** Translates between a raw DB record (or ORM model) and a domain object. Lives inside the concrete repository. Extracted to its own file when mapping logic is non-trivial.
- **Service File:** Consumes the repository interface. Contains business logic. Has no imports from the data layer.
- **Domain Object / Entity File:** The plain object the service works with. No ORM decorators.
- **DTO File:** Shapes for data entering or leaving the service boundary.
- **Test File:** Service unit tests use the in-memory repository. Repository integration tests verify the concrete implementation against a real (or containerised) database.
- **DI / Composition Root File:** The one place that imports both the service and the concrete repository and wires them together.

## Component Descriptions

- **Repository Interface:** The only thing the service knows about persistence. Changing the database means writing a new concrete class — the interface and the service are untouched.
- **Concrete Repository:** All database knowledge lives here: queries, transactions, connection handling, result mapping. Nothing outside this file should know what ORM or DB driver is in use.
- **Mapper:** Separates the translation concern from the query concern. A repository with complex mapping should extract this into a standalone mapper to keep both the query logic and the mapping logic readable.
- **In-Memory Repository:** A first-class implementation, not a mock. It should enforce the same invariants as the real repository (e.g., throw on duplicate inserts, return `null` for missing records). If a test passes against the in-memory repository, it should also pass against the real one.

## Example

Here is the full pattern applied to a user data access layer:

```text
src/
├── services/
│   └── user.service.js
├── domain/
│   └── user.entity.js
├── repositories/
│   ├── interfaces/
│   │   └── user-repository.interface.js
│   ├── postgres/
│   │   ├── user.postgres-repository.js
│   │   ├── user.mapper.js
│   │   └── user.schema.js
│   └── in-memory/
│       └── user.in-memory-repository.js
└── composition-root.js
```

### Interface (`user-repository.interface.js`)

```js
// Lives next to the service. No imports from any DB library.
class IUserRepository {
  async findById(id)           { throw new Error('Not implemented') }
  async findByEmail(email)     { throw new Error('Not implemented') }
  async save(user)             { throw new Error('Not implemented') }
  async delete(id)             { throw new Error('Not implemented') }
}
```

### Concrete repository (`user.postgres-repository.js`)

```js
// Implements IUserRepository. Imports pg/TypeORM/Knex — nothing outside this
// file should know we are using Postgres.
class UserPostgresRepository extends IUserRepository {
  async findById(id) {
    const row = await db('users').where({ id }).first()
    return row ? UserMapper.toDomain(row) : null
  }

  async save(user) {
    const row = UserMapper.toPersistence(user)
    await db('users').insert(row).onConflict('id').merge()
  }
}
```

### Mapper (`user.mapper.js`)

```js
class UserMapper {
  static toDomain(row) {
    return new User({ id: row.id, email: row.email, active: row.is_active })
  }

  static toPersistence(user) {
    return { id: user.id, email: user.email, is_active: user.active }
  }
}
```

### In-memory repository (`user.in-memory-repository.js`)

```js
// Used in tests. Behaves the same as the real repository for cases the service
// cares about. No DB connection needed.
class UserInMemoryRepository extends IUserRepository {
  constructor() { this._store = new Map() }

  async findById(id)       { return this._store.get(id) ?? null }
  async findByEmail(email) { return [...this._store.values()].find(u => u.email === email) ?? null }
  async save(user)         { this._store.set(user.id, user) }
  async delete(id)         { this._store.delete(id) }
}
```

### Service (`user.service.js`)

```js
// Depends only on the interface. No database imports.
class UserService {
  constructor(userRepository) {  // IUserRepository injected
    this._repo = userRepository
  }

  async getUserById(id) {
    const user = await this._repo.findById(id)
    if (!user) throw new UserNotFoundError(id)
    return user
  }
}
```

### Composition root (`composition-root.js`)

```js
// The only file that knows which concrete repository is in use.
const userRepository = new UserPostgresRepository(db)
const userService    = new UserService(userRepository)
```

### Test (`user.service.test.js`)

```js
// No database. Runs fast.
const repo    = new UserInMemoryRepository()
const service = new UserService(repo)
```
