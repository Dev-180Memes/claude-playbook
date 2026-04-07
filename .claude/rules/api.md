# API Design

## Paths

- Use lowercase, hyphen-separated words: `/user-profiles`, not `/userProfiles` or `/user_profiles`
- Use nouns for resources, never verbs: `/users`, not `/getUsers` or `/fetchUser`
- Use plural nouns for collections: `/users`, `/orders`, `/products`
- Nest resources to express ownership, but limit nesting to two levels:
  - Good: `/users/{userId}/orders`
  - Avoid: `/users/{userId}/orders/{orderId}/items/{itemId}/reviews`
- Use path parameters for resource identity: `/users/{userId}`
- Use query parameters for filtering, sorting, and pagination: `/users?role=admin&sort=createdAt&page=2`
- Do not encode actions in the path — use the HTTP method instead:
  - `DELETE /users/{userId}` not `POST /users/{userId}/delete`
  - `PATCH /users/{userId}` not `POST /users/{userId}/update`
- Version the API in the path prefix: `/v1/users`, `/v2/users`
- Trailing slashes are not significant — pick one style and enforce it consistently

## HTTP Methods

- `GET` — read a resource or collection; must be safe and idempotent
- `POST` — create a new resource or trigger a non-idempotent action
- `PUT` — replace a resource entirely (full update)
- `PATCH` — partial update of a resource
- `DELETE` — remove a resource; should be idempotent

## Status Codes

- `200 OK` — successful GET, PUT, PATCH
- `201 Created` — successful POST that created a resource; include a `Location` header pointing to the new resource
- `204 No Content` — successful DELETE or action with no response body
- `400 Bad Request` — invalid input from the client (validation errors, malformed body)
- `401 Unauthorized` — not authenticated; include a `WWW-Authenticate` header
- `403 Forbidden` — authenticated but not authorized
- `404 Not Found` — resource does not exist
- `409 Conflict` — request conflicts with current state (e.g., duplicate unique field)
- `422 Unprocessable Entity` — request is well-formed but semantically invalid
- `429 Too Many Requests` — rate limit exceeded; include `Retry-After` header
- `500 Internal Server Error` — unhandled server error; never leak stack traces to clients

## Request & Response

- Always return JSON; set `Content-Type: application/json`
- Use `camelCase` for JSON field names
- Wrap collections in a named key, not a bare array: `{ "users": [...], "total": 100 }`
- Include a consistent error shape for all error responses:
  ```json
  { "error": { "code": "USER_NOT_FOUND", "message": "No user with id 42 exists." } }
  ```
- Return the full updated resource on `PUT`/`PATCH`, not just a success flag
- Do not return internal database IDs directly if they expose implementation — use stable public identifiers
- Paginated responses should include metadata: `page`, `perPage`, `total`, `nextCursor` (if cursor-based)

## Versioning

- Version at the path level (`/v1/`) for REST APIs — it is explicit and easy to route
- Never make breaking changes within an existing version
- Deprecate old versions with a `Deprecation` response header before removing them
- Document the sunset date for deprecated versions

## General

- Idempotency: `PUT`, `DELETE`, and `GET` must be safe to retry; `POST` should accept an idempotency key for critical operations
- Avoid chatty APIs — design endpoints so clients can accomplish a task in as few round trips as possible
- Do not expose internal implementation details (table names, ORM errors, stack traces) in responses
- Rate limit all public endpoints and document the limits
