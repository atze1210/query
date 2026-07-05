---
name: write-query-feature
description: Add a new feature or fix a bug in query-core (QueryClient, QueryCache, QueryObserver, etc.). Use when the user wants to extend core query behaviour, add a new option, or patch a bug in `packages/query-core`.
license: MIT
---

# Write a query-core Feature

Add a new option, behaviour, or bug fix to `@tanstack/query-core`.

## When to use

- User wants to add a new `QueryClient` method or option.
- User wants to change how `QueryObserver`, `QueryCache`, or `MutationCache` behaves.
- User needs to fix a bug in the core package.

## Package layout

```
packages/query-core/src/
  query.ts             ← Query instance and QueryState
  queryCache.ts        ← QueryCache (stores and manages Query instances)
  queryClient.ts       ← QueryClient (public API surface)
  queryObserver.ts     ← QueryObserver (drives useQuery)
  infiniteQueryObserver.ts
  mutationCache.ts
  mutationObserver.ts
  mutation.ts
  notifyManager.ts     ← batches subscriber notifications
  focusManager.ts
  onlineManager.ts
  retryer.ts           ← retry/backoff logic
  types.ts             ← all public TypeScript types
  utils.ts             ← shared helpers (hashKey, partialMatchKey, …)
  index.ts             ← public barrel
  __tests__/           ← Vitest tests, mirror the src/ structure
```

## Steps

### 1. Identify the right file

| Change type | Start here |
|-------------|-----------|
| New `QueryClient` method or default option | `queryClient.ts` |
| New per-query option | `types.ts` (add the option), then `query.ts` or `queryObserver.ts` (consume it) |
| Change fetch / retry / cancel behaviour | `retryer.ts` or `query.ts` |
| Change how results are calculated | `queryObserver.ts` |
| Mutation change | `mutation.ts`, `mutationObserver.ts`, or `mutationCache.ts` |
| New public type only | `types.ts` |

### 2. Add the TypeScript types first

Open `types.ts` and add or extend the relevant interface/type. Keep the addition minimal — prefer union extensions over new interfaces. Use `OmitKeyof` when you need to omit keys from another type.

### 3. Implement the feature

Follow the existing patterns in the file:

- All state mutations on `Query` and `Mutation` go through `#dispatch` so subscribers are notified.
- Use `notifyManager.batch(() => { … })` when you need to fire multiple subscriber notifications atomically.
- Do **not** import from framework adapters — `query-core` must remain framework-agnostic.
- Keep new public methods on `QueryClient` narrow: accept a `QueryFilters` or `MutationFilters` argument where the operation is filter-based.

### 4. Export from the barrel if needed

If you've added a new public type or function, add it to `index.ts`.

### 5. Write tests

Add a test file in `src/__tests__/` that mirrors the source file (e.g. `queryClient.test.ts`).

Run the tests:

```bash
# type tests
pnpm --filter @tanstack/query-core test:types

# runtime tests
pnpm --filter @tanstack/query-core test:lib
```

Or from the repo root using nx:

```bash
npx nx run @tanstack/query-core:test:lib
```

### 6. Check framework adapters

If you changed a type or option that framework adapters consume, search for usages:

```bash
grep -r "yourNewOption\|yourNewMethod" packages/ --include="*.ts" --include="*.tsx"
```

Update any affected adapters.

### 7. Add a changeset

```bash
pnpm changeset
```

Select `@tanstack/query-core`. Choose `patch` for bug fixes, `minor` for new non-breaking features, `major` for breaking changes.

## Don't

- Don't add framework-specific code to `query-core`.
- Don't forget to export new public types from `index.ts`.
- Don't skip the type tests — they catch regressions that runtime tests miss.
- Don't run `pnpm run test:lib` from inside the package directory; always use `nx` from the repo root.
