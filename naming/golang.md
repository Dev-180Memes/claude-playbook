# Go Naming Conventions

- `camelCase` for unexported identifiers; `PascalCase` for exported ones
- Acronyms must be consistently cased: `userID`, `parseURL`, `HTTPClient` — not `userId`, `parseUrl`, `HttpClient`
- Interface names should describe behavior, typically ending in `-er`: `Reader`, `Writer`, `Handler`, `Stringer`
- Single-word, lowercase package names — not plural, no underscores: `user`, `auth`, `config` — not `users`, `user_service`
- Short variable names (`i`, `n`, `err`, `buf`, `ctx`) are idiomatic in short scopes; use descriptive names in longer functions
- Receiver variable names should be a one or two letter abbreviation of the type name, consistent across all methods: `func (u *User)`, not `func (this *User)` or `func (self *User)`
- Error variables should be named `err` for the primary error and `<type>Err` for domain errors: `validationErr`
