# Testing

## Philosophy

- Tests verify behaviour, not implementation — test what a unit does, not how it does it internally
- A test that breaks when you refactor without changing behaviour is a bad test
- Prefer fewer, well-targeted tests over many brittle ones
- Tests are first-class code — apply the same standards of clarity and maintainability

## Test Types & When to Use Them

- **Unit tests** — pure functions, domain logic, data transformations; fast, no I/O
- **Integration tests** — interactions between components: service + database, API handler + middleware; hit real dependencies where practical
- **End-to-end tests** — critical user journeys only; kept to a minimum due to cost and fragility
- Do not mock what you can run cheaply — prefer real implementations (in-memory DB, test containers) over mocks that diverge from production behaviour

## Structure & Naming

- Name tests to describe the behaviour being verified:
  - `returns 404 when user does not exist`
  - `sends a confirmation email after successful registration`
- Organise tests to mirror the source directory structure
- Group related tests with `describe`/`context` blocks scoped to the unit under test
- Each test should have one logical assertion — split tests that verify multiple independent behaviours

## Arrange–Act–Assert

- Structure every test as: set up the scenario, perform the action, assert the outcome
- Keep the arrange section minimal — if setup is complex, extract helpers or factories
- Avoid logic (loops, conditionals) in tests — they make failures hard to diagnose

## Mocks & Stubs

- Mock at the boundary of your system (HTTP clients, third-party APIs, email providers) — not within it
- Never mock the thing under test
- Prefer dependency injection over module-level monkey-patching so dependencies are easy to replace in tests
- Assert on mock calls only when the interaction itself is the behaviour being tested — do not assert on every call mechanically

## Test Data

- Use factories or builders to create test data — avoid copy-pasting large fixture objects across tests
- Use the minimum data required for the test — do not set fields the test does not care about
- Do not share mutable state between tests — each test must be independently runnable in any order

## Coverage

- Aim for coverage of all meaningful code paths, not a specific percentage
- 100% line coverage does not mean correct behaviour — tests must also assert the right outcomes
- Prioritise coverage of edge cases, error paths, and boundary conditions over happy-path lines

## CI & Reliability

- All tests must pass on CI before merging — never merge with known failing tests
- Tests must not depend on execution order, timing (`sleep`), or external network calls
- Use deterministic seeds for any randomness in tests
- Keep the test suite fast — slow tests discourage running them locally

## General

- Delete tests that no longer reflect real behaviour rather than commenting them out
- Fix flaky tests immediately — a test that sometimes fails is worse than no test because it erodes trust
- Do not test framework behaviour or third-party library internals — test your own code's use of them
