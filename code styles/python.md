# Python Code Style

## Formatting

- Follow PEP 8 for all formatting rules
- Use 4 spaces for indentation — never tabs
- Maximum line length of 88 characters (Black default)
- Use Black for auto-formatting and isort for import sorting

## Type Hints

- Always annotate function signatures with type hints (parameters and return types)
- Use `from __future__ import annotations` for forward references in older Python versions
- Use `X | None` over `Optional[X]` (Python 3.10+); use `Optional[X]` only when targeting earlier versions
- Use `TypeVar` and generics for reusable typed utilities
- Run `mypy` in strict mode to catch type errors statically

## Naming Conventions

- `snake_case` for variables, functions, methods, and modules
- `PascalCase` for classes
- `SCREAMING_SNAKE_CASE` for module-level constants
- Prefix private attributes/methods with a single underscore `_`
- Avoid double underscore `__` name mangling unless you specifically need to prevent subclass override

## Functions

- Keep functions small and focused on a single responsibility
- Use keyword arguments for calls with more than two parameters for clarity
- Avoid mutable default arguments (e.g., `def f(x=[])`) — use `None` and set inside the function
- Prefer returning explicit values over mutating arguments in place

## Classes

- Prefer dataclasses or Pydantic models over plain classes for data containers
- Use `__slots__` when creating many instances of a class to reduce memory overhead
- Do not use bare `except:` — always catch a specific exception type

## Imports

- Standard library first, then third-party, then local — each group separated by a blank line
- Never use wildcard imports (`from module import *`)
- Avoid circular imports by restructuring dependencies

## Error Handling

- Catch the most specific exception possible
- Use context managers (`with`) for resource management (files, locks, connections)
- Raise `ValueError` for invalid inputs, `TypeError` for wrong types — use built-in exceptions before defining custom ones
- Custom exceptions should inherit from a project-level base exception class

## General

- Prefer list/dict/set comprehensions over `map`/`filter` for simple transformations
- Use `pathlib.Path` over `os.path` for filesystem operations
- Use f-strings for string formatting; avoid `%` formatting and `.format()` where f-strings are clearer
- Do not rely on implicit `None` returns — be explicit with `-> None` annotations
