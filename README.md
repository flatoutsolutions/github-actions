# github-actions

Reusable GitHub Actions for the FlatOut Solutions platform stack.

## Actions

### [`setup`](./setup/)

Node.js + Yarn setup with layered dependency caching. Replaces the typical `setup-node` → `corepack enable` → `yarn install` boilerplate with a single step that caches `node_modules` directly (skipping `yarn install` entirely on cache hit).

**Cache layers:**

| Layer | Cached path                        | Cache key                 | Skips on hit            |
| ----- | ---------------------------------- | ------------------------- | ----------------------- |
| 1     | `node_modules`                     | `yarn.lock` hash          | `yarn install`          |
| 2     | `frontend/node_modules`            | `frontend/yarn.lock` hash | frontend `yarn install` |
| 3     | `convex/_generated` (configurable) | Convex source files hash  | `npx convex codegen`    |

**Inputs:**

| Input                        | Default                | Description                                        |
| ---------------------------- | ---------------------- | -------------------------------------------------- |
| `node-version`               | `22.x`                 | Node.js version                                    |
| `convex-codegen`             | `false`                | Run `npx convex codegen`                           |
| `convex-deploy-key`          | `''`                   | Convex deploy key (required if codegen is enabled) |
| `convex-dir`                 | `.`                    | Working directory for Convex codegen               |
| `convex-generated-path`      | `convex/_generated`    | Path to generated output (for caching)             |
| `convex-source-glob`         | `convex/**/*.ts`       | Source files glob (for cache key)                  |
| `convex-source-exclude-glob` | `convex/_generated/**` | Glob to exclude from source hash                   |
| `install-frontend`           | `false`                | Install `frontend/` dependencies                   |

**Usage (root Convex):**

```yaml
steps:
  - uses: actions/checkout@v6
  - uses: flatoutsolutions/github-actions/setup@v1
    with:
      convex-codegen: "true"
      convex-deploy-key: ${{ secrets.CONVEX_DEPLOY_KEY }}
```

**Usage (frontend Convex):**

```yaml
steps:
  - uses: actions/checkout@v6
  - uses: flatoutsolutions/github-actions/setup@v1
    with:
      install-frontend: "true"
      convex-codegen: "true"
      convex-deploy-key: ${{ secrets.CONVEX_DEPLOY_KEY }}
      convex-dir: frontend
      convex-generated-path: frontend/src/convex/_generated
      convex-source-glob: "frontend/src/convex/**/*.ts"
      convex-source-exclude-glob: "frontend/src/convex/_generated/**"
```

### [`changes`](./changes/)

Path-based change detection for monorepo workflows. Wraps [`dorny/paths-filter`](https://github.com/dorny/paths-filter) so one `detect-changes` job can gate many downstream jobs, instead of every repo re-implementing the same block.

**Inputs:**

| Input                  | Default  | Description                                                             |
| ---------------------- | -------- | ----------------------------------------------------------------------- |
| `filters`              | required | YAML mapping filter names to path globs (or a path to a `.yml` file)    |
| `base`                 | `''`     | Ref to compare against. Empty = paths-filter's default for the event    |
| `ref`                  | `''`     | Ref to compare. Empty = the triggering ref                              |
| `predicate-quantifier` | `some`   | `some` or `every` — how multiple patterns within one filter combine     |

**Output:** a single `changes` output — a **JSON array of the filter names that matched**.

There is one output rather than one per filter because a composite action must declare its outputs statically, while the filter set is supplied by the caller at runtime. Per-filter outputs would mean hard-coding every name any consumer might use.

**Usage:**

```yaml
detect-changes:
  runs-on: ubuntu-latest
  outputs:
    changes: ${{ steps.filter.outputs.changes }}
  steps:
    - uses: actions/checkout@v6
    - id: filter
      uses: flatoutsolutions/github-actions/changes@v1
      with:
        filters: |
          website:
            - 'website/**'
          server:
            - 'server/**'

deploy-website:
  needs: detect-changes
  if: contains(fromJSON(needs.detect-changes.outputs.changes), 'website')
  runs-on: ubuntu-latest
  steps: ...
```

The action does **not** check out for you — the caller owns checkout, so it can run against a tree the caller has already prepared.

## Versioning

Actions are versioned with git tags. Pin to a major version for stability:

```yaml
- uses: flatoutsolutions/github-actions/setup@v1 # recommended
- uses: flatoutsolutions/github-actions/setup@v1.2.0 # exact version
```
