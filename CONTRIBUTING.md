# Contributing

## Local Setup

Install dependencies before making changes:

```bash
npm install
```

The `prepare` script installs Husky hooks for the checkout.

## Pre-Commit Checks

Before each commit, `lint-staged` runs on staged files:

- JavaScript and TypeScript files run `eslint --fix` and `prettier --write`.
- JSON, CSS, Markdown, and config files run `prettier --write`.

Run the same staged-file checks manually with:

```bash
npm run lint:staged
```

## Commit Messages

Commit messages must follow the Conventional Commits format:

```text
type(optional-scope): short description
```

Accepted types are `build`, `chore`, `ci`, `docs`, `feat`, `fix`, `perf`, `refactor`, `revert`, `style`, and `test`.

Examples:

```text
feat: add wallet display
fix(auth): handle rejected connection
```
