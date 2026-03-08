# github-actions

Reusable GitHub Actions for the FlatOut Solutions platform stack.

## Actions

### [`setup`](./setup/)

Node.js + Yarn setup with layered dependency caching. Replaces the typical `setup-node` → `corepack enable` → `yarn install` boilerplate with a single step that caches `node_modules` directly (skipping `yarn install` entirely on cache hit).

**Cache layers:**

| Layer | Cached path         | Cache key                | Skips on hit         |
| ----- | ------------------- | ------------------------ | -------------------- |
| 1     | `node_modules`      | `yarn.lock` hash         | `yarn install`       |
| 2     | `convex/_generated` | Convex source files hash | `npx convex codegen` |

**Usage:**

```yaml
steps:
  - uses: actions/checkout@v6
  - uses: flatoutsolutions/github-actions/setup@v1
    with:
      convex-codegen: "true"
      convex-deploy-key: ${{ secrets.CONVEX_DEPLOY_KEY }}
```

**Inputs:**

| Input               | Default   | Description                                        |
| ------------------- | --------- | -------------------------------------------------- |
| `node-version`      | `22.21.1` | Node.js version                                    |
| `convex-codegen`    | `false`   | Run `npx convex codegen`                           |
| `convex-deploy-key` | `''`      | Convex deploy key (required if codegen is enabled) |

## Versioning

Actions are versioned with git tags. Pin to a major version for stability:

```yaml
- uses: flatoutsolutions/github-actions/setup@v1 # recommended
- uses: flatoutsolutions/github-actions/setup@v1.0.0 # exact version
```
