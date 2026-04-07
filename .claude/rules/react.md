# React

## Components

- One component per file; file name matches the component name in `PascalCase`
- Prefer function components — never write new class components
- Keep components small and focused on a single responsibility; if a component needs more than one screen of code, consider splitting it
- Co-locate styles, tests, and sub-components with the component they belong to
- Do not put business logic inside JSX — extract to a variable or function above the return statement

## Props

- Define prop types with TypeScript interfaces, not `PropTypes`
- Destructure props in the function signature: `function UserCard({ name, email }: UserCardProps)`
- Use explicit `children: React.ReactNode` when a component accepts children — do not use `React.FC`
- Boolean props that default to `false` should be written as flags: `<Button disabled />`, not `<Button disabled={true} />`
- Avoid prop drilling beyond two levels — lift state up or use context/state management

## Hooks

- Only call hooks at the top level — never inside conditions, loops, or nested functions
- Prefix custom hooks with `use`: `useAuthState`, `usePagination`
- Extract repeated stateful logic into a custom hook rather than duplicating it across components
- `useEffect` dependencies must be complete and accurate — do not silence linter warnings with comments
- Avoid `useEffect` for data transformations that can be derived directly from state or props

## State

- Keep state as local as possible — lift it only when two or more components genuinely need to share it
- Use `useState` for simple local state; use `useReducer` when state transitions have multiple cases or depend on previous state
- Do not store derived values in state — compute them from existing state during render
- Avoid storing copies of props in state — read from props directly

## Performance

- Do not optimise prematurely — `React.memo`, `useMemo`, and `useCallback` add complexity and should only be used when a measurable performance problem exists
- Use keys that are stable and unique when rendering lists — never use array indices as keys for lists that can reorder or change
- Lazy-load heavy components with `React.lazy` and `Suspense`

## Data Fetching

- Do not fetch inside `useEffect` for new code — prefer a data-fetching library (`React Query`, `SWR`) or framework-level loaders
- Always handle loading and error states explicitly — never assume a fetch will succeed
- Avoid waterfalls — fetch data in parallel where possible

## Structure

```
src/
  components/       # shared, reusable components
  features/         # feature-scoped components, hooks, and logic
  hooks/            # shared custom hooks
  pages/ (or app/)  # route-level components
  lib/              # third-party setup and utilities
```

## General

- Do not use indexes as React `key` props for dynamic lists
- Avoid inline object and function literals in JSX props that cause unnecessary re-renders: extract them above the JSX or memoize
- Keep JSX readable — if a conditional expression is complex, extract it into a named variable or component
- Do not suppress ESLint rules for `react-hooks/exhaustive-deps` — fix the underlying dependency issue
