# TypeScript Code Style

## Compiler Settings

- Always enable strict mode: `"strict": true` in `tsconfig.json`
- Enable `"noUncheckedIndexedAccess": true` to catch array/object access issues
- Enable `"exactOptionalPropertyTypes": true` for precise optional property handling

## Types

- Never use `any`. Use `unknown` when the type is truly unknown and narrow it explicitly
- Prefer `interface` for object shapes that may be extended; use `type` for unions, intersections, and aliases
- Use `readonly` for properties that should not be mutated
- Avoid type assertions (`as`) unless absolutely necessary — prefer type guards instead
- Use discriminated unions over optional properties to model mutually exclusive states
- Avoid `!` non-null assertions. Narrow with conditionals or use optional chaining instead

## Naming Conventions

- `PascalCase` for types, interfaces, classes, and enums
- `camelCase` for variables, functions, and methods
- `SCREAMING_SNAKE_CASE` for true constants (values that never change)
- Prefix interfaces with a meaningful name, not `I` (e.g., `UserRepository`, not `IUserRepository`)
- Boolean variables and functions should read as yes/no questions: `isLoading`, `hasError`, `canSubmit`

## Functions

- Always declare explicit return types on exported functions
- Prefer `function` declarations for top-level functions; arrow functions for callbacks and closures
- Keep functions small and single-purpose
- Avoid default exports — prefer named exports for better refactoring and discoverability

## Null & Undefined

- Use `undefined` for optional/absent values; avoid `null` unless interfacing with external APIs that require it
- Use optional chaining (`?.`) and nullish coalescing (`??`) instead of manual null checks

## Enums

- Prefer `const enum` for purely numeric/string constants to avoid runtime overhead
- Prefer a plain object `as const` union over enums when values are strings, for simpler interop

## Imports

- Use named imports; avoid `import *`
- Group imports: external packages first, then internal modules, separated by a blank line
- Do not use path aliases that obscure the actual file location

## Error Handling

- Always handle promise rejections — no floating promises
- Type caught errors as `unknown` and narrow them before use (do not assume `Error` shape)
- Prefer returning `Result`-style types (`{ data, error }`) over throwing in library/service code

## General

- No unused variables or imports (enforced by `noUnusedLocals` and `noUnusedParameters`)
- Prefer `for...of` over `.forEach` when you need `break`/`continue` or `await`
- Use `Array<T>` or `T[]` consistently — pick one per project and stick to it
