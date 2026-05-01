# Bit Component Management

The DAEN ecosystem uses [Bit](https://bit.dev) to version and distribute shared JS libraries across repositories. This document covers the setup, workflow, and constraints that apply to all repos.

## Scope

All shared components live under the **`desabeillesetnous.daen-js-shared`** Bit scope on [bit.cloud](https://bit.cloud/desabeillesetnous/daen-js-shared).

## Workspace locations

Each repo that uses Bit has a workspace at a specific root:

| Repo | Workspace root | `defaultDirectory` |
|---|---|---|
| `fb-admin` | `fb-admin/daen-config-cli/` | `daen-lib/{name}` |
| `daen-scout` | `daen-scout/` | `daen-lib/{name}` |
| `daen-fb-workers` | `daen-fb-workers/functions/` | `lib/{name}` |

All `bit` commands must be run from the workspace root of that repo.

## Component registry

| Component | Scope version | Canonical repo | Path in canonical repo |
|---|---|---|---|
| `daen-firebase` | `0.1.1` | `fb-admin` | `daen-config-cli/daen-lib/daen-firebase/` |
| `daen-objects` | `0.1.0` | `fb-admin` | `daen-config-cli/daen-lib/daen-objects/` |
| `daen-utils` | `0.0.2` | `daen-scout` | `daen-lib/daen-utils/` |
| `looped-carousel` | `0.0.4` | `daen-scout` | `daen-lib/looped-carousel/` |
| `snackbar` | `0.0.3` | `daen-scout` | `daen-lib/snackbar/` |

## How Bit is used here

Bit acts as a **version sync tool**, not a runtime module resolver. The distinction matters:

- `bit import` fetches the latest component source into a local subfolder (`lib/` or `daen-lib/`)
- At runtime, code imports components via **relative paths** — not via the npm package names Bit generates
- Bit's `node_modules/@desabeillesetnous/` symlinks are local dev artifacts only; they are never deployed

This keeps deployment bundles (especially GCP Cloud Functions) fully self-contained without requiring access to Bit's npm registry.

## Runtime import convention

Always use relative paths when importing shared components:

```js
// ✅ correct — works in all deployment contexts
const { POI } = require('../daen-objects');
const { geoloc } = require('../daen-utils');

// ❌ wrong — not deployed to GCP, will fail at runtime
const { POI } = require('@desabeillesetnous/daen-js-shared.daen-objects');
```

In `daen-fb-workers`, this convention is enforced by ESLint (`no-restricted-imports`).

## Logging in shared components

Shared components use `console` for logging — not project-specific loggers like `gc_logger`. This keeps them deployment-context-agnostic:

- Cloud Functions code uses `gc_logger` for structured GCP logging
- Shared lib components (`lib/daen-firebase`, etc.) use `console.warn` / `console.error`
- `gc_logger` is a local utility in `daen-fb-workers` — it is not a Bit component and must not be imported inside shared components

## Common workflows

### Pulling an update from scope

```bash
# From the workspace root of the consuming repo
bit import desabeillesetnous.daen-js-shared/daen-objects
```

This updates the local source files and `.bitmap`. Review changes before committing.

### Publishing a change from the canonical repo

```bash
# From the canonical repo's workspace root
bit tag daen-firebase --patch --message "fix: correct FieldValue usage"
bit export
```

Consuming repos then pull with `bit import`.

### Checking workspace status

```bash
bit status          # shows modified, new, and imported components
bit list            # shows all tracked components and their scope versions
```

### Local overloads

A repo may modify a component's source locally after `bit import` (e.g. to use relative imports instead of npm package names in fb-workers). `bit status` shows the component as modified. Only export back to scope if the change is generic enough to benefit all consumers. Project-specific modifications stay local.

## Scope migration history

The scope was migrated from `starlingpartners.daen-lib` to `desabeillesetnous.daen-js-shared` in May 2026. All `.bitmap` files and `workspace.jsonc` files reference the new scope. The old scope no longer exists.

## Troubleshooting

**`component objects are missing from the scope`**
The local `.bit` object store is out of sync. Run `bit import --objects <component-id>` to restore without touching source files. If the scope is empty for that component, the objects were never exported — tag and export from the canonical repo first.

**`your workspace has outdated objects`**
Run `bit import --objects` to pull the latest model from the remote scope.

**`nothing to tag`**
The component may be in an invalid state (missing objects). Clear the `.bit/` directory, run `bit init`, re-add the component with `bit add`, then tag.
