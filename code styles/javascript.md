# JavaScript Code Style

## Tooling

- Use ESLint with a consistent ruleset (e.g., `eslint:recommended` + `plugin:import/recommended`)
- Use Prettier for auto-formatting; do not hand-tune formatting rules that Prettier manages
- Enable `"use strict"` in non-module scripts; it is implicit in ES modules

## Variables

- Always use `const` by default; use `let` only when reassignment is required
- Never use `var` — its function-scoped hoisting causes subtle bugs
- Declare variables at the narrowest scope possible

## Functions

- Prefer arrow functions for callbacks and inline expressions
- Use named `function` declarations for top-level functions to aid stack traces and hoisting
- Avoid implicit returns in arrow functions with side effects — be explicit
- Default parameters are preferred over `|| fallback` patterns

## Naming Conventions

- `camelCase` for variables, functions, and methods
- `PascalCase` for classes and constructor functions
- `SCREAMING_SNAKE_CASE` for true constants
- Boolean variables should read as yes/no questions: `isReady`, `hasPermission`

## Modules

- Use ES module syntax (`import`/`export`) — avoid `require()` in new code
- Prefer named exports over default exports for better refactoring support
- Do not re-export everything from a barrel file unless intentional — it hurts tree-shaking

## Null & Undefined

- Use `===` for all equality checks; never `==`
- Use optional chaining (`?.`) and nullish coalescing (`??`) to handle absent values safely
- Prefer `undefined` over `null` for unset values in application code

## Async

- Use `async`/`await` over raw Promise chains for readability
- Always `try/catch` around `await` calls or handle rejections at the call site
- Never leave floating promises unhandled — use `void` explicitly if intentionally fire-and-forget

## Error Handling

- Throw `Error` objects, never plain strings or objects
- Catch errors at the boundary closest to where recovery is possible
- Include context in error messages so they are actionable when logged

## General

- Use array methods (`map`, `filter`, `reduce`) over imperative loops for transformations
- Avoid mutating function arguments — return new values instead
- Do not use `eval` or `new Function` — they open XSS and code injection risks
- Keep files focused: one primary export per file where practical
