# Configuration Reference

Exhaustive syntax reference for all haloy.yaml fields. Consult this when users need advanced features beyond the basics covered in SKILL.md.

## File Discovery

Haloy searches the current directory for configuration in this order:

1. `haloy.yaml`
2. `haloy.yml`
3. `haloy.json`
4. `haloy.toml`

Custom location: `haloy deploy --config /path/to/config.yaml`

**Casing conventions:**
- YAML/TOML: `snake_case` (e.g., `health_check_path`)
- JSON: `camelCase` (e.g., `healthCheckPath`)

## Environment Variables

### Four Forms

**Plain value:**
```yaml
env:
  - name: "NODE_ENV"
    value: "production"
```

**From local environment variable (`from.env`):**
```yaml
env:
  - name: "DATABASE_URL"
    from:
      env: "PRODUCTION_DATABASE_URL"
```
The deployer must set `PRODUCTION_DATABASE_URL` locally before running `haloy deploy`.

**From secret provider (`from.secret`):**
```yaml
env:
  - name: "DATABASE_PASSWORD"
    from:
      secret: "onepassword:production-db.password"
```
Requires a matching `secret_providers` block (see Secret Providers section).

**Build argument (`build_arg`):**
```yaml
env:
  - name: "NODE_ENV"
    value: "production"
    build_arg: true
```
Passes the variable as a Docker build argument in addition to a runtime env var.

### Variable Interpolation

Use `${VAR_NAME}` syntax inside `value:` strings to reference other env vars. Only `${VAR_NAME}` is supported (not bare `$VAR`). Does not apply to `from:`-sourced values. Undefined references remain as literal text. Circular dependencies trigger deployment errors.

```yaml
env:
  - name: "DB_PASSWORD"
    from:
      secret: "onepassword:db-creds.password"
  - name: "DATABASE_URL"
    value: "postgres://admin:${DB_PASSWORD}@db:5432/myapp"
```

### Environment File Loading Order

1. `.env` (current directory)
2. `.env.local` (overrides `.env`)
3. `.env.{target}` (target-specific, e.g., `.env.production`)
4. `~/.config/haloy/.env`

### Multi-Target Env Merging

Target-specific variables merge with base configuration. Duplicates are overridden by the target:

```yaml
env:
  - name: "LOG_LEVEL"
    value: "info"
targets:
  production:
    env:
      - name: "LOG_LEVEL"
        value: "warn"
```

## Image Configuration

### String Shorthand

A plain string works anywhere an image is expected:

```yaml
image: "nginx:alpine"
```

For `images` map (named image references):
```yaml
images:
  db: "postgres:18"
  cache: "redis:7"
```

### Object Form

```yaml
image:
  repository: "my-app"        # required - Docker image name
  tag: "v1.2.3"               # defaults to "latest"
  build: true                  # build locally instead of pulling
  source: "registry"           # "registry" or "local"
  registry:                    # auth for private registries
    server: "ghcr.io"
    username: "my-user"
    password:
      from:
        secret: "onepassword:ghcr-creds.password"
  build_config:
    context: "."               # build directory (default: current dir)
    dockerfile: "Dockerfile"   # path to Dockerfile
    platform: "linux/amd64"    # target architecture
    args:                      # build-time variables
      NODE_ENV: "production"
    push: "server"             # "registry" or "server"
  history:
    strategy: "local"          # "local", "registry", or "none"
    keep: 3                    # number of images to retain
```

### Registry Authentication

Supports GitHub Container Registry (GHCR), Docker Hub, Azure Container Registry, AWS ECR, and self-hosted registries. Credentials can be provided directly or referenced from environment variables and secret providers. Haloy can auto-detect the registry server from the repository name.

### History / Rollback Strategies

- **local** (default): Preserves tagged images locally with configurable retention count
- **registry**: Relies on immutable registry tags matching specified patterns
- **none**: Disables rollback capability (used by database/service presets)

## Secret Providers

### 1Password Setup

```yaml
secret_providers:
  onepassword:
    source-name:
      account: "optional-account-id"
      vault: "VaultName"
      item: "ItemName"
```

### Reference Format

```
"onepassword:<source-name>.<field-name>"
```

### Full Example

```yaml
secret_providers:
  onepassword:
    production-db:
      vault: "Production"
      item: "Database Credentials"

env:
  - name: "DB_PASSWORD"
    from:
      secret: "onepassword:production-db.password"
  - name: "DATABASE_URL"
    value: "postgres://admin:${DB_PASSWORD}@db:5432/myapp"
```

### Where Secrets Can Be Used

- Environment variables (`from.secret`)
- Registry authentication (`password.from.secret`)
- API tokens
- Build arguments (combined with `build_arg: true`)

### Prerequisites

- 1Password CLI (`op`) must be installed and authenticated
- Vaults and items must exist with the referenced fields

### Validation

```bash
haloy validate-config                       # checks without displaying secrets
haloy validate-config --show-resolved-config # displays resolved values
```

## Volumes

### Named Volumes (Recommended)

```yaml
volumes:
  - "app-data:/app/data"
  - "postgres-data:/var/lib/postgresql/data"
```

### Bind Mounts

```yaml
volumes:
  - "/var/app/data:/app/data"
  - "/home/user/logs:/app/logs:ro"
```

**Absolute paths are mandatory** for bind mounts. Relative paths resolve relative to the daemon container's filesystem, not the local machine, creating data loss risks.

### Mount Modifiers

- `:ro` - read-only access
- `:rw` - read-write (default)
- `:z` - SELinux shared labeling
- `:Z` - SELinux private labeling

## Deploy Hooks

### Per-Target Hooks

```yaml
pre_deploy:
  - "npm run db:migrate"
  - "npm run db:seed"
post_deploy:
  - "curl -X POST https://hooks.slack.com/notify"
```

- `pre_deploy`: Executes before target deployment on the local machine
- `post_deploy`: Executes after target deployment on the local machine

### Global Hooks (Top-Level Only)

```yaml
global_pre_deploy:
  - "echo 'Starting deployment'"
global_post_deploy:
  - "echo 'All targets deployed'"
```

- `global_pre_deploy`: Runs once before all target deployments
- `global_post_deploy`: Runs once after all target deployments

## Naming Strategy

```yaml
naming_strategy: "dynamic"   # default
```

- **dynamic** (default): Containers get unique names per deployment
- **static**: Container names are fixed. Requires `deployment_strategy: "replace"` and does not support multiple replicas

## Protected Targets

```yaml
protected: true    # default: false
```

- Protected targets are skipped by `haloy deploy --all`
- Deploy explicitly with `haloy deploy -t targetname`
- Force inclusion with `--include-protected`

## Presets

### Database Preset

`preset: "database"` automatically sets:
- `deployment_strategy: "replace"`
- `naming_strategy: "static"`
- `protected: true`
- `image.history.strategy: "none"`

### Service Preset

`preset: "service"` automatically sets:
- `deployment_strategy: "replace"`
- `naming_strategy: "static"`
- `image.history.strategy: "none"`

## API Token

```yaml
api_token:
  from:
    env: "MY_HALOY_TOKEN"
```

Usually not needed. Haloy reads the token from `~/.config/haloy/client.yaml` by default.

## All Fields Summary

### Global-Only Fields (Top-Level)

| Field | Type | Purpose |
|-------|------|---------|
| `targets` | object | Defines multiple deployment targets |
| `images` | map | Named image references (strings or objects) |
| `secret_providers` | object | Configure secret providers |
| `global_pre_deploy` | array | Commands before all deployments |
| `global_post_deploy` | array | Commands after all deployments |

### Target-Level Fields

| Field | Type | Default |
|-------|------|---------|
| `name` | string | Required for single-deployment |
| `server` | string | Haloy server URL |
| `image` | string/object | Docker image reference |
| `port` | string/integer | "8080" |
| `replicas` | integer | 1 |
| `domains` | array | Domain routing config |
| `health_check_path` | string | "/" |
| `env` | array | Environment variables |
| `volumes` | array | Volume mounts |
| `deployment_strategy` | string | "rolling" |
| `naming_strategy` | string | "dynamic" |
| `protected` | boolean | false |
| `preset` | string | "database" or "service" |
| `pre_deploy` | array | Commands before this target deploys |
| `post_deploy` | array | Commands after this target deploys |
| `network` | string | "haloy" |
| `api_token` | object | Auth config (usually not needed) |
