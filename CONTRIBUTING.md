# Contributing to StellarKit UI

Thanks for helping improve StellarKit UI. This guide explains how to set up the project, choose issues, make focused changes, and submit pull requests that are easy to review.

## Local Development

1. Fork the repository and clone your fork.
2. Install dependencies:

```bash
npm install
```

3. Start the development server:

```bash
npm run dev
```

4. Open `http://localhost:3000` and verify the app loads.

The main development commands are:

```bash
npm run dev
npm run build
npm run lint
```

Run `npm run lint` before opening a pull request. For UI, routing, or configuration changes, also run `npm run build` when possible.

## Project Structure

The current repository is a single Next.js app:

- `app/` contains App Router pages, layouts, and global styles.
- `public/` contains static assets served by Next.js.
- `package.json` contains npm scripts and dependency versions.
- `eslint.config.mjs`, `tsconfig.json`, `postcss.config.mjs`, and `next.config.ts` configure tooling.

Some issues may describe a future monorepo layout such as `packages/ui` or demo apps. If those folders do not exist on `main`, keep your change aligned with the current structure or explain the mismatch in the issue before implementing broad scaffolding.

## Picking Issues

Before starting:

- Check that the issue is still open.
- Read the full acceptance criteria.
- Search open pull requests for the issue number and related feature name.
- Prefer small, focused issues that can be verified locally.
- Ask for clarification when the issue references files or APIs that do not exist on `main`.

If you claim an issue, leave a concise comment with your intended approach and avoid claiming multiple issues you cannot complete promptly.

## Branch Naming

Use short, descriptive branch names:

```text
docs/contributing-29
fix/wallet-loading-state-20
feat/network-switcher-7
test/account-balance-25
```

Recommended prefixes:

- `docs/` for documentation-only changes.
- `fix/` for bug fixes.
- `feat/` for new behavior or components.
- `test/` for test-only work.
- `chore/` for tooling or maintenance.

## Commit Messages

Use an imperative, scoped commit message:

```text
Add contributing guide
Fix wallet loading reset
Document network switcher usage
```

Keep unrelated work in separate commits or separate pull requests. Avoid formatting-only changes outside the files required by the issue.

## Writing Tests

Run the checks that match your change:

- Documentation-only changes usually need spelling/link review and do not require build changes.
- Component, hook, or utility changes should include focused tests when a test setup exists.
- Bug fixes should include a regression test or a clear manual verification note.
- If no test command exists for the affected area, state that in the pull request and describe the manual checks you ran.

When adding a new test framework or script, keep the setup minimal and document the new command in `package.json` or the relevant README.

## Pull Request Process

Before opening a pull request:

1. Sync with `main`.
2. Re-run relevant checks.
3. Confirm your diff only includes files needed for the issue.
4. Write a clear PR description with summary, validation, and issue link.

Use this PR body shape:

```markdown
## Summary
- What changed
- Why it changed

## Validation
- npm run lint
- npm run build

Closes #123
```

If a command fails because of an existing repository setup problem, include the exact command and failure summary instead of omitting it.

## Issue Labels

Common labels:

- `documentation`: README, guides, examples, and written reference material.
- `good first issue`: a scoped task suitable for new contributors.
- `bug`: broken or incorrect behavior.
- `feature`: new behavior, components, hooks, or app functionality.
- `security`: changes involving validation, signing, data handling, or safe defaults.
- `test`: unit, integration, accessibility, visual, or regression test work.
- `Stellar Wave`: issues associated with the Stellar Wave contribution program.

Labels are a guide, not a substitute for the issue body. The acceptance criteria in the issue are the source of truth.

## Review Etiquette

- Keep the conversation focused and respectful.
- Respond to review comments with either a code update or a short explanation.
- Do not force-push unrelated rewrites after review starts.
- If you discover an issue is already solved or mismatched with `main`, say so and close or pause your work rather than submitting a no-op PR.
