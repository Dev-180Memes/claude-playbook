# Git Rules

## Branching

- Never push directly to `main` — all changes go through a feature branch
- Branch names should follow the pattern: `<type>/<short-description>` (e.g., `feat/user-auth`, `fix/login-redirect`, `chore/update-deps`)
- Keep branches short-lived — merge or close them promptly to avoid long-running divergence
- Delete branches after they are merged

## Commit Messages

Use the Conventional Commits format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

**type** — one of:

- `feat` — new feature
- `fix` — bug fix
- `docs` — documentation only
- `style` — formatting, whitespace (no logic change)
- `refactor` — code change that is neither a fix nor a feature
- `test` — adding or updating tests
- `chore` — build process, dependency updates, tooling

**scope** — the area of the codebase being changed (e.g., `auth`, `api`, `database`). Omit if the change is truly global.

**subject** — a brief imperative summary of what the commit does, present tense, no period at the end. Maximum 50 characters.

**body** — explain the *why*, not the *what*. Wrap at 72 characters per line. Omit if the subject line is self-explanatory.

**footer** — references to issues or PRs (e.g., `Closes #42`, `Refs #17`).

### Examples

```
feat(auth): add OAuth2 login with Google

Users can now sign in with their Google account instead of creating
a separate password. Reduces friction during onboarding.

Closes #88
```

```
fix(api): return 404 when user record is not found
```

## Pull Requests

- One PR per logical change — do not bundle unrelated fixes
- PR title should match the commit message format: `type(scope): subject`
- Include a short description of what changed and why
- Link the relevant issue in the PR description
- Ensure CI passes before requesting review
- Do not force-push to a branch that others are reviewing

## Merging

- Prefer squash merge for feature branches to keep `main` history linear and readable
- Use merge commits only when preserving the full branch history is intentional
- Never rebase shared branches — only rebase your own local branches before opening a PR
- Do not merge your own PR without at least one review, unless the team has agreed otherwise

## General

- Keep commits atomic — each commit should represent one coherent change that compiles and passes tests on its own
- Do not commit generated files, build artifacts, or secrets — use `.gitignore`
- Resolve merge conflicts by understanding both sides; do not blindly accept one side
- Use `git stash` to preserve in-progress work before switching context, not temporary commits
