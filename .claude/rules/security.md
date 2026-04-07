# Security

## Input Validation

- Validate all input at the system boundary — never trust data from clients, external APIs, or environment variables without validation
- Reject invalid input early and return a clear `400` error — do not attempt to sanitize or coerce malformed data silently
- Validate both type and shape; also enforce business-rule constraints (e.g., length limits, allowed character sets)
- Use an allowlist approach for validation where possible — define what is permitted, not what is forbidden

## SQL & Injection

- Always use parameterised queries or prepared statements — never concatenate user input into SQL strings
- ORMs are not a substitute for parameterised queries when using raw query escape hatches — treat those the same as raw SQL
- Never expose raw database error messages to clients — they can leak schema information

## Authentication & Authorisation

- Authenticate every request that accesses protected data — do not rely solely on obscurity or client-side guards
- Authorise at the resource level, not just the route level — check that the authenticated user is allowed to access the specific record
- Use short-lived tokens (JWTs, session tokens) and implement refresh token rotation
- Hash passwords with a strong, salted algorithm (`bcrypt`, `argon2`) — never store plaintext or weakly hashed passwords
- Invalidate sessions server-side on logout — do not rely on deleting the client cookie alone

## Secrets Management

- Never hardcode secrets, API keys, or credentials in source code or commit them to version control
- Use environment variables for secrets in local development; use a secrets manager (Vault, AWS Secrets Manager, etc.) in production
- Rotate secrets regularly and immediately after any suspected exposure
- Audit `.gitignore` before committing — ensure `.env`, credential files, and key files are always excluded

## Transport & Data

- Enforce HTTPS everywhere — no HTTP in production
- Set secure HTTP headers: `Strict-Transport-Security`, `X-Content-Type-Options`, `X-Frame-Options`, `Content-Security-Policy`
- Never log sensitive data: passwords, tokens, full payment details, PII — log correlation IDs instead
- Encrypt sensitive data at rest for anything that would cause harm if the database were breached

## XSS & CSRF

- Never inject raw user-controlled content into HTML without proper escaping
- Use framework-provided escaping mechanisms — do not write custom sanitisation
- Set `SameSite=Strict` or `SameSite=Lax` on cookies to mitigate CSRF
- Use CSRF tokens for state-changing form submissions if not relying on `SameSite` cookies

## Dependencies

- Keep dependencies up to date — regularly audit with `npm audit`, `pip audit`, `cargo audit`, or equivalent
- Do not add dependencies for trivial functionality that can be implemented safely in a few lines
- Pin dependency versions in production builds and review changelogs before upgrading

## Error Handling

- Never expose stack traces, internal paths, or system information in API error responses
- Log detailed errors server-side with context; return only a safe, generic message to the client
- Use a consistent error response format so clients cannot infer implementation details from varying error shapes

## General

- Follow the principle of least privilege — services, users, and API keys should only have the permissions they actually need
- Avoid `eval`, `exec`, dynamic code execution, or shell command construction from user input
- Review code that touches authentication, payment, or PII with extra scrutiny before merging
