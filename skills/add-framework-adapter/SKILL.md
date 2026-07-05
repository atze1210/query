---
name: add-framework-adapter
description: Add a new framework adapter package (e.g. for a new UI framework) to TanStack Query. Use when the user asks "how do I add a new adapter", "how do I port query to <framework>", or wants to create a new `packages/<framework>-query` package.
license: MIT
---

# Add a Framework Adapter to TanStack Query

Port TanStack Query to a new UI framework by building a thin adapter package on top of `@tanstack/query-core`.

## When to use

- User wants to create `packages/<framework>-query` for a framework not yet supported.
- User asks "how do I port query to X?".

## Overview

All framework adapters share the same pattern:

1. Wrap `QueryClient` and `QueryCache` from `@tanstack/query-core` with framework-idiomatic providers.
2. Implement reactive hooks/composables/stores using the framework's reactivity primitives that delegate to `QueryObserver`, `InfiniteQueryObserver`, `MutationObserver`, and `QueriesObserver`.
3. Re-export the core types users need.

Look at `packages/vue-query`, `packages/svelte-query`, or `packages/solid-query` as the closest reference implementations (they are the leanest adapters).

## Steps

### 1. Create the package directory

```bash
mkdir packages/<framework>-query
cd packages/<framework>-query
```

Copy the `package.json`, `tsconfig.json`, `tsconfig.legacy.json`, and `vite.config.ts` from an existing adapter (e.g. `packages/vue-query`) and update:
- `name` → `@tanstack/<framework>-query`
- `description`
- `peerDependencies` → add the new framework, remove the old one
- `dependencies` → point `@tanstack/query-core` at `workspace:*`

### 2. Register the package in the monorepo

Add the new package to `pnpm-workspace.yaml` (it is usually covered by the existing `packages/*` glob — verify).

Add a project entry to `nx.json` if needed (usually auto-detected).

### 3. Implement the adapter in `src/`

Required files (use react-query or vue-query as a reference):

| File | Purpose |
|------|---------|
| `index.ts` | Public barrel — re-export everything the user needs |
| `queryClient.ts` | Framework-wrapped `QueryClient` — needed when the framework's reactivity system must own the client instance (e.g. Vue's `provide`/`inject`, Svelte stores). Omit this file if the framework lets you pass the client directly as a prop or function argument (as React's `QueryClientProvider` does). |
| `queryOptions.ts` | Re-export or extend `queryOptions` from query-core |
| `infiniteQueryOptions.ts` | Same for infinite queries |
| `mutationOptions.ts` | Same for mutations |
| `useQuery.ts` | Main hook/composable — delegates to `QueryObserver` |
| `useInfiniteQuery.ts` | Delegates to `InfiniteQueryObserver` |
| `useMutation.ts` | Delegates to `MutationObserver` |
| `useQueries.ts` | Delegates to `QueriesObserver` |
| `useIsFetching.ts` | Convenience hook using `queryClient.isFetching()` |
| `types.ts` | Adapter-specific type aliases and augmentations |

The central pattern for a reactive hook:

1. Create an observer instance: `new QueryObserver(queryClient, options)`.
2. Subscribe to the observer's `subscribe(listener)` callback and update local reactive state.
3. Return the current `observer.getCurrentResult()` as the hook's return value.
4. Unsubscribe when the component/composable is destroyed.

### 4. Write tests

Mirror the test structure from an existing adapter:

```
src/__tests__/
  useQuery.test.ts
  useInfiniteQuery.test.ts
  useMutation.test.ts
  useQueries.test.ts
```

Run your tests:

```bash
npx nx run @tanstack/<framework>-query:test:lib
```

### 5. Add documentation

Create `docs/framework/<framework>/` and add at minimum:
- `overview.md`
- `quick-start.md`
- `installation.md`

Register the new pages in `docs/config.json`.

### 6. Add a changeset

```bash
pnpm changeset
```

Select `@tanstack/<framework>-query` and write a short description. This is a new package so choose `minor`.

## Don't

- Don't copy-paste business logic from query-core into the adapter — call the observer APIs instead.
- Don't add a new adapter without tests; CI will reject it.
- Don't add the framework as a regular dependency — it must be a `peerDependency`.
