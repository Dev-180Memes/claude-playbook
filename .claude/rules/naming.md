# Naming Conventions

## General Principles

- Names should be self-documenting — a reader should understand what something is or does without needing a comment
- Avoid abbreviations unless they are universally understood in the domain (e.g., `id`, `url`, `api`, `ctx`)
- Do not use single-letter names except for loop indices (`i`, `j`) or well-established math/algorithm conventions
- Names should reflect intent and domain language, not implementation details (e.g., `userRepository`, not `userDbHelper`)
- Boolean names should read as yes/no questions: `isActive`, `hasPermission`, `canEdit`
- Avoid noise words that add no meaning: `data`, `info`, `manager`, `handler`, `helper` — be specific about what the thing actually does

## Files & Directories

- Use lowercase with hyphens for file and directory names: `user-profile.ts`, `auth-middleware/`
- Match the file name to its primary export: a file exporting `UserProfile` should be named `user-profile.ts`
- Test files should mirror the source file name: `user-profile.test.ts` for `user-profile.ts`

## Functions & Methods

- Use verb phrases that describe the action: `getUser`, `validateEmail`, `parseConfig`
- Getters return a value — prefix with `get`, `fetch`, or `find` based on whether there is I/O involved
- Predicates return booleans — prefix with `is`, `has`, `can`, `should`
- Event handlers should be prefixed with `on` or `handle`: `onSubmit`, `handleClick`
- Avoid vague names like `process`, `execute`, `run`, `do` — say what specifically is being done

## Variables & Constants

- Variable names should describe the value they hold, not the type: `users` not `userArray`
- Constants that are truly fixed should be `SCREAMING_SNAKE_CASE` in most languages
- Prefer full words over truncated forms: `response` not `res`, `request` not `req`, `error` not `err` — unless the language community has a strong idiom otherwise (see language-specific rules)

## Language-Specific Rules

For language-specific naming conventions, see:

- [naming/typescript.md](../../naming/typescript.md)
- [naming/javascript.md](../../naming/javascript.md)
- [naming/python.md](../../naming/python.md)
- [naming/golang.md](../../naming/golang.md)
- [naming/rust.md](../../naming/rust.md)
