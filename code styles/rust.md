# Rust Code Style

## Formatting & Linting

- All code must be formatted with `rustfmt` — run it on every save or via CI
- Use `clippy` with `#![deny(clippy::all)]` (or `clippy::pedantic` for stricter checks) and fix all warnings
- Do not suppress warnings with `#[allow(...)]` without a documented reason

## Naming Conventions

- `snake_case` for variables, functions, methods, modules, and crate names
- `PascalCase` for types, traits, and enum variants
- `SCREAMING_SNAKE_CASE` for constants and statics
- Trait method names should describe what the implementor does: `Display`, `Iterator`, `From`

## Ownership & Borrowing

- Prefer borrowing (`&T`, `&mut T`) over cloning unless ownership transfer is semantically correct
- Clone only at boundaries where it is necessary — document why if it is non-obvious
- Avoid `Rc<RefCell<T>>` in hot paths; it trades compile-time safety for runtime panics
- Prefer `Arc<Mutex<T>>` over raw `unsafe` sharing for concurrent state

## Error Handling

- Use `Result<T, E>` for all fallible operations — never `panic!` for recoverable errors
- Define domain-specific error types using `thiserror` for libraries
- Use `anyhow` for application-level error propagation where specific error types are not needed
- Use the `?` operator to propagate errors; avoid `.unwrap()` and `.expect()` outside tests and prototypes
- When `.expect()` is used, the message must describe what invariant was violated, not just what failed

## Types & Traits

- Use newtypes (`struct Meters(f64)`) to make domain concepts type-safe and prevent unit confusion
- Implement standard traits (`Display`, `Debug`, `From`, `Into`, `Default`) where they make sense
- Prefer `impl Trait` in function arguments for ergonomics; use generics when bounds need to be named or reused
- Use `#[derive(...)]` for common trait implementations rather than writing them manually

## Structs & Enums

- Use enums to model state machines and mutually exclusive variants — avoid boolean flags that encode hidden state
- Keep struct fields private by default; expose via methods to maintain invariants
- Use the builder pattern for structs with many optional fields

## Concurrency

- Leverage the type system — `Send` and `Sync` bounds catch sharing bugs at compile time
- Prefer message passing with channels (`std::sync::mpsc` or `tokio::sync`) over shared mutable state
- Keep critical sections (locks held) as short as possible

## Modules & Visibility

- Use `pub(crate)` for items that are internal to the crate but shared across modules
- Keep `pub` surface area minimal — expose only what external consumers need
- Organize code into modules that reflect logical boundaries, not file size

## General

- Write doc comments (`///`) for all public items — include a brief summary and at least one example
- Write unit tests in a `#[cfg(test)] mod tests` block in the same file
- Use `cargo test`, `cargo clippy`, and `cargo fmt --check` in CI
- Avoid `unsafe` unless interfacing with FFI or implementing low-level abstractions — isolate it and document the invariants that make it sound
