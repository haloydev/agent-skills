---
name: haloy-config
description: Create a haloy.yaml configuration file for deploying applications with haloy. Use when the user says "create haloy config", "add haloy.yaml", "configure for haloy", "set up haloy deployment", or "prepare for haloy". This skill creates deployment configurations without docker-compose.
license: MIT
compatibility: Requires a haloy server. Works with any Dockerized application.
metadata:
  author: haloydev
  version: "1.0.0"
---

# Haloy Configuration

Create `haloy.yaml` configuration files for deploying applications with [haloy](https://haloy.dev).

## Important: Scope Check

Before proceeding, verify the user's request aligns with this skill:

**This skill is for:**
- Creating a `haloy.yaml` configuration file
- Single-target deployments (one server, one app)
- Applications that already have a Dockerfile or use a pre-built image

**This skill is NOT for:**
- Creating Dockerfiles (use the `dockerize` skill instead)
- Multi-target deployments (staging/production) - users can expand the config later
- docker-compose setups or Kubernetes manifests

If no Dockerfile exists and the user needs one, inform them:

> "No Dockerfile found in this project. Haloy deploys Docker containers, so you'll need a Dockerfile first. Would you like me to create one using the `dockerize` skill?"

## How It Works

1. **Check for Dockerfile** - Verify the project can be containerized
2. **Check for configured haloy servers** - Look for existing server configurations
3. **Detect the project type** by examining:
   - `package.json` with framework dependencies (Next.js, TanStack Start, Express, etc.)
   - `pyproject.toml`, `requirements.txt`, `Pipfile` (Python/Django/FastAPI)
   - `go.mod` (Go)
   - `Cargo.toml` (Rust)
   - `Gemfile` (Ruby/Rails)
   - `composer.json` (PHP/Laravel)
4. **Infer defaults** for app name, port, and health check path
5. **Ask the user** for server (from configured list or manual entry) and domain
6. **Generate haloy.yaml** with appropriate configuration
7. **Provide next steps** for validation and deployment

## Project Detection

Detect the framework to determine the default port:

| Framework | Indicator | Default Port |
|-----------|-----------|--------------|
| Next.js | `next` in package.json dependencies | 3000 |
| TanStack Start | `@tanstack/react-start` in package.json | 3000 |
| Express/Node.js | `express` in package.json or generic Node | 3000 |
| Vite | `vite` in package.json | 5173 (dev) / 3000 (prod) |
| Django | `django` in requirements.txt or pyproject.toml | 8000 |
| FastAPI | `fastapi` in requirements.txt or pyproject.toml | 8000 |
| Flask | `flask` in requirements.txt or pyproject.toml | 5000 |
| Go | `go.mod` present | 8080 |
| Rust | `Cargo.toml` present | 8080 |
| Ruby/Rails | `rails` in Gemfile | 3000 |
| PHP/Laravel | `laravel` in composer.json | 8000 |
| Default | Unknown framework | 8080 |

Also check the Dockerfile for `EXPOSE` directives which override framework defaults.

## Decision Flow

Use defaults unless a critical value is missing. Only ask the user when necessary.

### App Name
- **First choice**: `name` field in `package.json`
- **Second choice**: Project folder name
- **Ask if**: Name contains invalid characters or is generic (e.g., "app", "project")

### Server URL

Check for configured servers in `~/.config/haloy/client.yaml`:

```yaml
# Example client.yaml structure
servers:
  example.haloy.dev:
    token_env: HALOY_API_TOKEN_example_HALOY_DEV
  prod.mycompany.com:
    token_env: HALOY_API_TOKEN_PROD
```

**Decision flow:**

1. **If servers found**: Present them as options and let the user choose. The server keys (e.g., `example.haloy.dev`) are the values to use in the config.
   > "I found these configured haloy servers. Which one would you like to deploy to?"
   > - example.haloy.dev
   > - prod.mycompany.com
   > - Enter a different server

2. **If no servers found**: Check if haloy CLI is installed by running `which haloy` or `haloy --version`
   - **If haloy is installed**: Ask for the server URL manually
     > "What is your haloy server URL? (e.g., haloy.yourserver.com)"
   - **If haloy is not installed**: Inform the user they need to install and configure the CLI first
     > "No haloy servers configured. To deploy with haloy, you'll need to install the CLI and add a server. See: https://haloy.dev/docs/client-installation"

### Domain
- **Always ask** - Required for web applications
- Example prompt: "What domain should this app be accessible at? (e.g., myapp.example.com)"
- If user says "none" or "skip", omit the domains section

### Domain Aliases
- **Ask after domain**: Once the user provides a domain, ask if they want to add any aliases
- Example prompt: "Would you like to add any domain aliases? (e.g., www.myapp.example.com)"
- Common alias: `www.` prefix of the main domain
- If user says "none" or "skip", omit the aliases

### DNS Reminder
After gathering domain information, remind the user to configure DNS:

> "Remember to configure DNS with your domain provider. Point your domain(s) to your haloy server's IP address using an A record, or use a CNAME record if your server has a domain name."

### Port
- **Use framework default** from the table above
- **Check Dockerfile** for `EXPOSE` directive which takes precedence
- **Ask only if**: Multiple ports exposed or no clear default

### Health Check Path
- **Search for existing endpoints**: `/health`, `/healthz`, `/api/health`, `/_health`
- **If found**: Use the existing health endpoint
- **If not found**: Use `/` (root path)
- Do NOT ask - haloy will use `/` by default which works for most apps

## Configuration Reference

Haloy supports two modes:

1. **Single Deployment**: No `targets` defined, root configuration is the deployment target
2. **Multi-Target**: `targets` defined, root acts as base/default configuration

### Minimal Configuration (Single Deployment)

```yaml
name: "my-app"
server: "haloy.yourserver.com"
```

### With Domain and Port

```yaml
name: "my-app"
server: "haloy.yourserver.com"
domains:
  - domain: "my-app.example.com"
    aliases:
      - "www.my-app.example.com"
port: "3000"
health_check_path: "/health"
```

### With Environment Variables

```yaml
name: "my-app"
server: "haloy.yourserver.com"
domains:
  - domain: "my-app.example.com"
port: "3000"
env:
  - name: "NODE_ENV"
    value: "production"
  - name: "DATABASE_URL"
    value: "postgres://user:pass@db:5432/myapp"
```

### With Volumes (for persistent data)

```yaml
name: "my-app"
server: "haloy.yourserver.com"
domains:
  - domain: "my-app.example.com"
volumes:
  - "app-data:/app/data"
```

### Multi-Target Example

```yaml
# Base settings inherited by all targets
image:
  repository: "my-app"
  tag: "latest"
port: "3000"

targets:
  production:
    server: "prod.haloy.com"
    domains:
      - domain: "my-app.com"
    replicas: 2
  staging:
    server: "staging.haloy.com"
    domains:
      - domain: "staging.my-app.com"
    replicas: 1
```

### Full Reference

For all configuration options, see: https://haloy.dev/docs/configuration-reference

Key options:
- `name` - Application name (required for single deployment)
- `server` - Haloy server URL (required)
- `image` - Docker image configuration (repository, tag)
- `domains` - Array of domain objects with `domain` and optional `aliases`
- `port` - Container port (default: "8080")
- `health_check_path` - Health check endpoint (default: "/")
- `env` - Environment variables as name/value pairs
- `volumes` - Volume mounts for persistent data
- `replicas` - Number of container instances (default: 1)
- `deployment_strategy` - "rolling" (default) or "replace"
- `targets` - Define multiple deployment targets (multi-target mode)
- `preset` - Apply preset configuration ("database" or "service")

## Output Format

After gathering information, create the `haloy.yaml` file and provide:

1. **The haloy.yaml file** - Written to the project root
2. **Update .dockerignore** - Add `haloy.yaml` to `.dockerignore` if not already present. This improves Docker layer caching since changes to `haloy.yaml` won't invalidate the build cache.
3. **Validation command**:
   ```bash
   haloy validate-config
   ```
4. **Deployment command**:
   ```bash
   haloy deploy
   ```
5. **If haloy CLI is not installed**, show installation options:
   ```bash
   # Shell script
   curl -fsSL https://sh.haloy.dev/install-haloy.sh | sh

   # Homebrew
   brew install haloydev/tap/haloy

   # npm/pnpm/bun
   npm install -g haloy
   pnpm add -g haloy
   bun add -g haloy
   ```

## Example Interactions

### With Configured Servers

**User**: "Create a haloy config for this project"

**Agent**:
1. Checks for Dockerfile - found
2. Checks `~/.config/haloy/client.yaml` - finds configured servers
3. Detects Next.js from package.json
4. Infers app name "my-nextjs-app" from package.json
5. Detects port 3000 from framework
6. Finds `/api/health` endpoint in the codebase

**Agent asks**:
> "I found these configured haloy servers. Which one would you like to deploy to?"
> - example.haloy.dev
> - prod.mycompany.com

**User**: "example.haloy.dev"

**Agent asks**:
> "What domain should this app be accessible at?"

**User**: "myapp.example.com"

**Agent asks**:
> "Would you like to add any domain aliases? (e.g., www.myapp.example.com)"

**User**: "yes, www.myapp.example.com"

**Agent creates** `haloy.yaml`:
```yaml
name: "my-nextjs-app"
server: "example.haloy.dev"
domains:
  - domain: "myapp.example.com"
    aliases:
      - "www.myapp.example.com"
port: "3000"
health_check_path: "/api/health"
```

**Agent provides next steps**:
> Your `haloy.yaml` has been created.
>
> **DNS Setup**: Remember to configure DNS with your domain provider. Point `myapp.example.com` and `www.myapp.example.com` to your haloy server's IP address using A records.
>
> To deploy:
> 1. Validate the config: `haloy validate-config`
> 2. Deploy: `haloy deploy`

### Without Configured Servers

**User**: "Create a haloy config"

**Agent**:
1. Checks for Dockerfile - found
2. Checks `~/.config/haloy/client.yaml` - no servers configured
3. Checks if haloy CLI is installed - not found

**Agent responds**:
> "No haloy servers configured. To deploy with haloy, you'll need to install the CLI and add a server first. See: https://haloy.dev/docs/client-installation"
>
> Once installed, run `haloy server add` to configure your server, then run this skill again.
