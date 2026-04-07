# Python Naming Conventions

- `snake_case` for variables, functions, methods, module names, and package names
- `PascalCase` for classes
- `SCREAMING_SNAKE_CASE` for module-level constants
- Prefix private attributes and methods with a single underscore: `_cache`, `_validate`
- Avoid double underscore `__` name mangling unless preventing subclass override is specifically required
- Test functions must be prefixed with `test_`: `test_returns_none_when_user_not_found`
- Type alias names follow `PascalCase`: `UserId = int`, `ResponseData = dict[str, Any]`
