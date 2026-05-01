# CLAUDE.md - DAEN Ecosystem

Guidance for working across the DAEN-SCOUT (Beefree) ecosystem of interconnected repositories.

## Ecosystem Overview

DAEN-SCOUT is a multi-repository ecosystem with four main components:

| Repository | Purpose | Technology |
|---|---|---|
| **daen-scout** | Mobile client application | React Native / Expo |
| **daen-fb-workers** | Backend Cloud Functions & task orchestration | Firebase JS SDK / Node.js |
| **fb-admin** | Administrative tools and CLI utilities | Node.js CLI / TypeScript |
| **daen-docs** | Architecture, data model, and ecosystem documentation | Markdown / Documentation |

### Repository Interactions

```
┌─────────────────────────────────────────────────────────────┐
│  User (iOS/Android)                                         │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│  daen-scout (Mobile App)                                    │
│  ├─ React Native screens & components                       │
│  ├─ Redux state management                                  │
│  └─ Firebase client SDK integration                         │
└─────────────────────────────────────────────────────────────┘
         ↓ API Calls         ↓ Realtime Sync
┌─────────────────────────────────────────────────────────────┐
│  Firebase Backend Services                                  │
│  ├─ Firestore (business objects)                            │
│  ├─ Realtime Database (transitive data)                     │
│  ├─ Authentication                                          │
│  ├─ Cloud Storage                                           │
│  └─ Pub/Sub (job scheduling)                                │
└─────────────────────────────────────────────────────────────┘
         ↑
┌─────────────────────────────────────────────────────────────┐
│  daen-fb-workers (Cloud Functions)                          │
│  ├─ Scheduled functions (task orchestration)                │
│  ├─ Database triggers (Firestore/RTDB events)               │
│  ├─ Worker implementations (business logic)                 │
│  └─ Task queue management (buffer + scheduled)              │
└─────────────────────────────────────────────────────────────┘
         ↑
┌─────────────────────────────────────────────────────────────┐
│  fb-admin & daen-config-cli                                 │
│  └─ Administrative operations & bulk data management         │
└─────────────────────────────────────────────────────────────┘

Documentation: daen-docs (architectural guides, data model, dev conventions)
```

## Shared Development Environments

All repositories use the same four-tier environment system:

| Environment | Firebase Project | Purpose | Client Access |
|---|---|---|---|
| **dev** | `bsc-dev-7a548` (scout) or `bsc-dev-7a548` (functions) | Integration & testing | Developers only |
| **sandbox** | `bsc-sandbox-9fbeb` | Testing & QA | Internal testers |
| **staging** | `dsc-staging-eu` | Pre-production validation | Beta users & pilots |
| **live** | `dsc-live-eu` | Production environment | End users |

Each environment has:
- Independent Firebase project
- Separate database and storage
- Environment-specific credentials and configuration
- Isolated user data and analytics

**Environment selection mechanism varies by repository:**
- **daen-scout:** `DAEN_TARGET` environment variable (loads .env files)
- **daen-fb-workers:** Firebase CLI with `.firebaserc` (project aliasing)
- **fb-admin:** Service account key selection (interactive menu)
See [Configuration Management](#configuration-management) below for details.

## Shared Firebase Data Structure

### Firestore Collections (Business Objects)

Core collections exist in Firestore across all environments:

- **`POIs`** - Active points of interest (business data)
  - Used by: daen-scout (display on map), daen-fb-workers (generate tiles, manage lifecycle)
  - Indexed by: location, category, status

- **`POIs_attic`** - Archived/deleted POIs (historical data)
  - Used by: daen-fb-workers (archival triggers)

- **`tiles_view`** - Materialized POI tiles for efficient map clustering
  - Used by: daen-scout (map rendering)
  - Maintained by: daen-fb-workers (POI tile triggers)

- **`users`** - User profiles and metadata
  - Used by: daen-scout (auth, profile), daen-fb-workers (user triggers)

- **`sequences`** - Auto-incrementing counters for various entities
  - Used by: daen-fb-workers (POI ID generation)

### Realtime Database Paths (Transitive Data)

RTDB stores frequently-changing and task data:

- **`buffer`** - Immediate task queue (no throttling, instant execution)
  - Used by: daen-fb-workers (immediate worker dispatch)
  - Purpose: Time-critical operations

- **`tasks`** - Scheduled task queue (throttled execution, timestamp-ordered)
  - Used by: daen-fb-workers (task scheduler, worker dispatch)
  - Purpose: Bulk operations with write contention management

- **`logs`** - Task execution logs and results
  - Used by: daen-fb-workers (audit trail)

- **`feedbacks`** - User feedback awaiting processing
  - Used by: daen-scout (submit feedback), daen-fb-workers (process)

- **`subs`** - User subscriptions and notification triggers
  - Used by: daen-scout (manage subs), daen-fb-workers (emit notifications)

- **`feeds/{feedId}`** - Event logs per feed source
  - Used by: daen-fb-workers (feed triggers)

## Shared Libraries

Ecosystem-wide libraries are versioned as [Bit](https://bit.dev) components under the **`desabeillesetnous.daen-js-shared`** scope. Each repo has a Bit workspace (`workspace.jsonc` + `.bitmap`) that tracks which version of each component it uses.

> **Full workflow details:** see [`dev framework/Bit Component Management.md`](dev%20framework/Bit%20Component%20Management.md)

### Component registry

| Component | Description | Canonical repo | Consumers |
|---|---|---|---|
| **`daen-firebase`** | Firebase SDK helpers, POI persistence, push notifications | `fb-admin/daen-config-cli` | `daen-fb-workers` |
| **`daen-objects`** | Business object definitions (POI, User, Subscription, Feed) | `fb-admin/daen-config-cli` | all repos |
| **`daen-utils`** | Geolocation helpers, validation, formatting | `daen-scout` | `daen-fb-workers`, `fb-admin` |
| **`looped-carousel`** | Carousel UI component | `daen-scout` | `daen-scout` |
| **`snackbar`** | Snackbar UI component | `daen-scout` | `daen-scout` |

### Design principles

- **Canonical source**: each component has one authoritative repo. Changes flow outward via `bit export`, not by editing copies.
- **Runtime resolution**: repos always import shared components using **relative paths** (`require('../daen-objects')`), never npm package names (`@desabeillesetnous/...`). This keeps the GCP Functions deployment self-contained.
- **Bit as sync tool**: Bit handles versioning and distribution. It does not change how modules resolve at runtime.
- **Local overloads**: a repo may modify a component locally after `bit import` for project-specific needs. `bit status` will show it as modified. Only export back if the change is generic.

### Quick reference

```bash
# Pull latest version of a component (run from workspace root)
bit import desabeillesetnous.daen-js-shared/daen-objects

# Publish a change from the canonical repo
bit tag daen-firebase --patch --message "fix: ..."
bit export
```

## Development Standards

### Git Workflow

**Branch Strategy:**
- `live` - Production release branch
- `beta` - Beta/testing branch
- `dev` - Main development branch (protected, default target for PRs)
- Feature branches: `<type>/DSC-##-<description>` (for daen-scout features)
  - `<type>/` prefix: `feat/`, `fix/`, `refactor/`, `docs/`, `chore/`, etc.
  - `DSC-##`: Ticket number from issue tracker
  - Example: `feat/DSC-123-poi-search-improvements`

### Commit Messages

Follow **Conventional Commits** specification across all repositories:

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation changes
- `style` - Code style/formatting
- `refactor` - Code refactoring
- `test` - Test additions/modifications
- `chore` - Build, dependencies, tooling
- `perf` - Performance improvements
- `ci` - CI/CD configuration
- `build` - Build system changes

**Scopes** (context-specific):
- For daen-scout: `(map)`, `(auth)`, `(nav)`, `(store)`, `(ui)`
- For daen-fb-workers: `(worker)`, `(trigger)`, `(queue)`, `(poi)`
- For fb-admin: `(cli)`, `(config)`, `(data)`

**Example commits:**
```
feat(map): add POI search with location filtering
fix(auth): resolve token refresh race condition
chore(deps): update firebase-admin to 11.9.0
docs: update Firebase Realtime Database schema
```

### Naming Conventions

**Code Level** (consistent across JavaScript/TypeScript):
- **Component/Class files**: `PascalCase` (e.g., `MapScreen.jsx`, `Worker.js`)
- **Utility/non-component files**: `camelCase` (e.g., `firebaseHelpers.js`, `taskScheduler.js`)
- **Functions/Methods**: `camelCase` (e.g., `processTask()`, `calculateDistance()`)
- **Private methods**: `_camelCase` prefix (e.g., `_validateInput()`)
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `MAX_BATCH_SIZE`, `TASK_TIMEOUT_MS`)
- **Types/Interfaces**: `PascalCase` (e.g., `POI`, `TaskDefinition`, `Worker`)

**Directory Level**:
- **Feature directories** (containing domain logic): `PascalCase` (e.g., `/Map/`, `/Parameters/`, `/Authentication/`)
- **Service/utility directories**: `lowercase` (e.g., `/api/`, `/utils/`, `/lib/`, `/components/`)
- **Configuration directories**: `camelCase` (e.g., `/buildconfig/`, `/appconfig/`)
- **Data/content directories**: `lowercase` with hyphens (e.g., `/data/`, `/emulator-seed/`)

**Database Paths**:
- **Firestore collections**: `PascalCase` (e.g., `POIs`, `POIs_attic`, `users`)
- **RTDB paths**: `lowercase` (e.g., `buffer`, `tasks`, `logs`, `feeds`)

### Code Patterns & Principles

**Shared Patterns:**
- Use functional programming where possible
- Prefer pure functions over stateful logic
- Handle errors explicitly (no silent failures)
- Log significant operations and errors
- Use structured logging (JSON format when possible)

**Async Operations:**
- Use async/await syntax (modern, readable)
- Handle promise rejections explicitly
- Chain promises for complex flows

**Error Handling:**
- Catch and log all errors with context
- Provide meaningful error messages
- Use error boundaries in UI code
- Include error codes for API responses

**Testing:**
- Write tests for critical business logic
- Use integration tests for Firebase interactions (prefer over mocks)
- Test error cases, not just happy paths

## Configuration Management

### Environment Selection per Repository

**daen-scout (Mobile App):**
Uses `DAEN_TARGET` environment variable to select configurations:
```bash
DAEN_TARGET=dev    # Loads .env_dev, dev Firebase config
DAEN_TARGET=staging # Loads .env_staging, staging Firebase config
DAEN_TARGET=live    # Loads .env_live, live Firebase config
```
- Configuration files: `.env.dist`, `.env_dev`, `.env_staging`, `.env_live`
- Dynamic config: `app.config.js` processes environment variables based on `DAEN_TARGET`
- Build profiles: `eas.json` specifies DAEN_TARGET per profile

**daen-fb-workers (Backend Cloud Functions):**
Uses Firebase CLI project selection (not DAEN_TARGET):
```bash
firebase use dev      # Selects dev project (bsc-dev-7a548)
firebase use staging  # Selects staging project (dsc-staging-eu)
firebase use live     # Selects live project (dsc-live-eu)
```
- Project mapping: `.firebaserc` defines dev/sandbox/staging/live projects
- Centralized config: `fb-worker-config.js` (not environment-specific)
- Deployment: `firebase deploy` to selected project

**fb-admin (Admin CLI):**
Uses service account keys for project access:
- Service accounts: `keys/{project-id}-admin.json` (gitignored)
- Project selection: Interactive menu prompts user to select key file
- Admin SDK: Initializes with selected service account credentials

### Secrets & Credentials

**Never commit:**
- Firebase private keys
- API credentials
- Personal access tokens
- Database URLs with credentials

**Storage:**
- Use Firebase project configuration files (auto-managed)
- Use environment-specific build config files
- Use `.gitignore` to prevent accidental commits

## Critical Integration Points

### Firebase Security

- **Firestore rules** - Enforce data access policies (in each repo's `/firestore.rules`)
- **RTDB rules** - Enforce database access (in daen-fb-workers `/database.rules.json`)
- **Cloud Functions** - Run with admin SDK, bypass security rules
- **Client SDK** - Respects Firestore/RTDB rules

### Cross-Repo Dependencies

- **daen-scout** → daen-fb-workers: Triggers backend functions via Firestore writes and button clicks
- **daen-fb-workers** → daen-scout: Updates Firestore/RTDB that client listens to; sends push notifications
- **fb-admin** → daen-fb-workers + daen-scout: Direct database management and bulk operations
- All → **daen-docs**: Reference architecture and conventions

### Deployment Coordination

When deploying across repos:
1. Deploy daen-fb-workers first (backend functions must be ready)
2. Deploy daen-scout (mobile clients update automatically via OTA or app store)
3. Update daen-docs if architecture/patterns change
4. Notify fb-admin users if admin operations need adjustment

## When to Reference This File

Check the root CLAUDE.md when:
- You need ecosystem context or high-level architecture
- You're working across multiple repositories
- You need to understand naming conventions or commit standards
- You're learning about Firebase data structures
- You're setting up environments or deployment

Refer to **repository-specific** CLAUDE.md for:
- Repository architecture and directory structure
- Development commands and build processes
- Technology stack and frameworks
- Repository-specific code patterns and conventions
- Critical files and integration points in that repo

For detailed technical documentation, see **daen-docs**.
