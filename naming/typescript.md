# TypeScript Naming Conventions

- `PascalCase` for types, interfaces, classes, enums, and enum variants
- `camelCase` for variables, functions, methods, and parameters
- `SCREAMING_SNAKE_CASE` for true constants (values that never change at runtime)
- Do not prefix interfaces with `I` — use a meaningful name: `UserRepository`, not `IUserRepository`
- Generic type parameters should be a single uppercase letter (`T`, `K`, `V`) or a descriptive `PascalCase` word (`TItem`, `TKey`) when context requires clarity
- React component names must be `PascalCase`; hooks must be prefixed with `use`: `useAuthState`
- Event handler props on components should be prefixed with `on`: `onClick`, `onFormSubmit`
- Avoid prefixing types with the word "Type" or "Interface": `User` not `UserType`, `UserInterface`
