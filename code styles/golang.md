# Go Code Style

## Formatting

- All code must be formatted with `gofmt` (or `goimports`) — no exceptions
- Use `golangci-lint` with a standard config for static analysis
- Line length is not formally enforced but keep lines readable; wrap at ~100 characters

## Naming Conventions

- `camelCase` for unexported identifiers; `PascalCase` for exported ones
- Acronyms should be consistently cased: `userID`, `parseURL`, `HTTPClient` — not `userId`, `parseUrl`, `HttpClient`
- Interface names should describe behavior, typically ending in `-er`: `Reader`, `Stringer`, `Handler`
- Short variable names (`i`, `n`, `err`, `buf`) are idiomatic in short scopes; use descriptive names in longer functions
- Package names should be lowercase, single words, and not plural (e.g., `user`, not `users` or `userPkg`)

## Error Handling

- Always check and handle errors — never discard with `_` unless the reason is explicitly commented
- Return errors as the last return value; wrap with context using `fmt.Errorf("doing X: %w", err)`
- Define sentinel errors with `errors.New` at the package level for errors callers need to check
- Use `errors.Is` and `errors.As` for error inspection — never string-match error messages

## Functions & Methods

- Keep functions short and focused — if a function needs a comment to explain its sections, split it
- Accept interfaces, return concrete types (enables flexibility without over-abstracting)
- Avoid named return values except in short functions where they genuinely aid clarity
- Use value receivers for small structs; pointer receivers for structs that are mutated or large

## Structs & Interfaces

- Define interfaces at the point of use (consumer side), not the implementation side
- Keep interfaces small — prefer single-method interfaces where possible
- Embed types to compose behavior, not to fake inheritance
- Use struct tags consistently for serialization (`json`, `db`, etc.)

## Concurrency

- Do not share memory by communicating — communicate by sharing channels
- Always document which goroutines own which data
- Use `sync.Mutex` for simple shared state; channels for coordination and signaling
- Ensure all goroutines can terminate — avoid goroutine leaks with context cancellation

## Imports

- Use `goimports` to manage import grouping automatically: stdlib, then external, then internal
- Never use dot imports (`. "pkg"`) — they pollute the namespace and obscure origin
- Alias imports only to avoid name collisions, not for convenience

## General

- Prefer table-driven tests for functions with multiple input/output cases
- Use `context.Context` as the first parameter for any function that does I/O or may be cancelled
- Avoid `init()` functions — prefer explicit initialization in `main` or constructors
- Do not use `panic` for normal error flow — reserve it for truly unrecoverable programmer errors
