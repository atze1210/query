---
name: contribute
description: Set up TanStack Query for local development, run tests, add a changeset, and submit a pull request. Use when the user asks how to contribute, set up the repo, run tests, or submit a PR.
license: MIT
---

# Contribute to TanStack Query

Get from a fresh clone to a merged pull request.

## When to use

- User asks "how do I contribute?", "how do I set up the repo?", or "how do I run the tests?".
- User wants to fix a bug or add a feature and needs the full dev workflow.

## Steps

### 1. Set up the environment

```bash
nvm use                          # switch to the Node version in .nvmrc
corepack enable && corepack prepare  # ensure correct pnpm version
pnpm install                     # install all workspace dependencies
```

> TanStack Query uses symlink-based config files. Develop on Linux, macOS, or WSL — not native Windows.

### 2. Build all packages

```bash
pnpm build:all
```

### 3. Start watch mode (rebuilds on change)

```bash
pnpm run watch
```

Leave this running while you edit. Examples and tests will pick up your local changes automatically.

### 4. Make your changes

- Core logic lives in `packages/query-core/src/`.
- Framework adapters live in `packages/<framework>-query/src/`.
- Write or update tests alongside your changes in the `__tests__/` directory of the affected package.
- Update or add documentation in `docs/` if the change is user-visible. Remember to update `docs/config.json` if you add a new doc page.

### 5. Run tests

Run tests for a **specific package** from the repo root (always use `nx` — never `pnpm test` inside a package folder):

```bash
npx nx run @tanstack/query-core:test:lib
npx nx run @tanstack/react-query:test:lib
```

Run **all tests**:

```bash
pnpm run test
```

Run **type tests** for query-core:

```bash
pnpm --filter @tanstack/query-core test:types
```

### 6. Lint and format

```bash
pnpm run lint
pnpm run format
```

### 7. Add a changeset (required for package version bumps)

```bash
pnpm changeset
```

Follow the prompts to select the affected packages and write a short changelog description. Commit the generated file in `.changeset/`. Skip this step if your change only affects docs, examples, or tests.

### 8. Commit and push

Follow [Conventional Commits](https://www.conventionalcommits.org/). Be especially careful with `feat!` or `fix!` for breaking changes.

### 9. Open a pull request

Submit against `main`. Maintainers squash-merge and edit the commit message as needed.

## Don't

- Don't run `pnpm run test:lib` inside an individual package folder — it causes cross-package dependency failures.
- Don't forget `docs/config.json` when adding a new documentation page.
- Don't open a PR without a changeset if your change affects a published package.
