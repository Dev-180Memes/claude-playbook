# Rust Naming Conventions

- `snake_case` for variables, functions, methods, modules, and crate names
- `PascalCase` for types, traits, structs, enums, and enum variants
- `SCREAMING_SNAKE_CASE` for constants (`const`) and statics (`static`)
- Trait names should describe what an implementor is or does: `Display`, `Iterator`, `From`, `Into`
- Lifetime names should be short and meaningful: `'a` for generic lifetimes; `'static`, `'conn`, `'req` for named ones
- Avoid prefixing types with their kind: `Config` not `ConfigStruct`, `Error` not `ErrorEnum`
- Newtype wrappers should be named for the domain concept they enforce: `Meters(f64)`, `UserId(u64)` — not `WrappedF64`
