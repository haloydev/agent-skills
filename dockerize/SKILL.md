---
name: dockerize
description: Create a Dockerfile for the current project to deploy with haloy. Use when the user says "dockerize", "create a Dockerfile", "containerize my app", "make this deployable", or "prepare for haloy". This skill creates optimized, production-ready Dockerfiles without docker-compose.
license: MIT
compatibility: Requires Docker knowledge. Works with any application that can be containerized.
metadata:
  author: haloydev
  version: "1.0.0"
---

# Dockerize for Haloy

Create production-ready Dockerfiles optimized for deployment with [haloy](https://haloy.dev).

## Important: Scope Check

Before proceeding, verify the user's request aligns with this skill:

**This skill is for:**
- Creating a single Dockerfile for haloy deployment
- Containerizing applications without docker-compose
- Preparing apps for single-container production deployment

**This skill is NOT for:**
- Multi-container setups requiring docker-compose
- Local development environments with multiple services
- Kubernetes manifests or other orchestration

If the user's request involves docker-compose, multiple containers, or complex orchestration, inform them:

> "This skill creates single Dockerfiles for haloy deployment. Haloy works like docker-compose but for production, handling orchestration through its own configuration. If you need multiple services, each should have its own Dockerfile and be defined as separate services in your `haloy.yaml`. Would you like me to create a Dockerfile for a specific service instead?"

## How It Works

1. **Detect the project type** by examining:
   - `package.json` with `@tanstack/react-start` (TanStack Start)
   - `package.json` with `next` (Next.js)
   - `package.json` (Node.js/JavaScript/TypeScript)
   - `bun.lockb`, `pnpm-lock.yaml`, `package-lock.json`, or `yarn.lock` (Node package manager)
   - `requirements.txt`, `pyproject.toml`, `Pipfile` (Python)
   - `uv.lock`, `poetry.lock`, or `[tool.poetry]` in `pyproject.toml` (Python package manager)
   - `go.mod` (Go)
   - `Cargo.toml` (Rust)
   - `Gemfile` (Ruby)
   - `pom.xml`, `build.gradle` (Java)
   - `composer.json` (PHP)
   - Other indicators

2. **Analyze the application** to determine:
   - Build process and dependencies
   - Runtime requirements
   - Entry point / start command
   - Required environment variables
   - Static assets or build outputs

3. **Check for health endpoint** and ask user about creating one if missing (see Health Check section below)

4. **Create an optimized Dockerfile** following best practices:
   - Multi-stage builds to reduce image size
   - Appropriate base images (Alpine when possible)
   - Proper layer ordering for cache efficiency
   - Non-root user for security
   - HEALTHCHECK instruction pointing to the health endpoint

5. **Provide haloy configuration guidance** if no `haloy.yaml` exists

## Health Check Endpoint

A `/health` endpoint is strongly recommended for all haloy deployments. Before creating the Dockerfile, check if the application has an existing health endpoint.

### Why Health Checks Matter

Haloy uses health checks to:
- **Zero-downtime deployments**: New containers must pass health checks before receiving traffic
- **Auto-recovery**: Unhealthy containers are automatically restarted
- **Deployment validation**: Deployments fail fast if the app cannot start properly

Without a health check, haloy cannot verify your application is actually working, which can lead to routing traffic to broken containers.

### Check for Existing Health Endpoint

Search for existing health endpoints:
- `/health`, `/healthz`, `/api/health`, `/_health`
- Look for route definitions returning status 200 or `{ status: "ok" }`

### Ask the User

If no health endpoint exists, ask the user:

> "Your application doesn't appear to have a health check endpoint. Haloy uses health checks for zero-downtime deployments and auto-recovery. Would you like me to create a `/health` endpoint?"

If the user agrees, create a minimal health endpoint appropriate for the framework:

**TanStack Start** (`src/routes/health.tsx`):
```tsx
import { createFileRoute } from "@tanstack/react-router";

export const Route = createFileRoute("/health")({
  server: {
    handlers: {
      GET: async () => {
        return Response.json({ status: "ok" });
      },
    },
  },
});
```

**Express/Node.js**:
```javascript
app.get('/health', (req, res) => {
  res.json({ status: 'ok' });
});
```

**Next.js** (`app/health/route.ts` or `pages/api/health.ts`):
```typescript
export async function GET() {
  return Response.json({ status: 'ok' });
}
```

**FastAPI**:
```python
@app.get("/health")
def health():
    return {"status": "ok"}
```

**Go (net/http)**:
```go
http.HandleFunc("/health", func(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    w.Write([]byte(`{"status":"ok"}`))
})
```

### Health Check Best Practices

- Return quickly (avoid database queries in the basic health check)
- Return HTTP 200 for healthy, non-200 for unhealthy
- Keep the response minimal: `{"status": "ok"}` is sufficient
- Place at a consistent path: `/health` is the convention

## Decision Flow

Use defaults unless a critical value is missing. Only ask the user when needed.

- **Health endpoint**: Ask only if no existing health route is found.
- **Start command**: Use existing `scripts.start` or framework default. Ask if no clear entry point exists.
- **Port**: Use common defaults (3000 for Node, 8000 for Python) unless a config file or env var specifies a port.
- **Package manager**: Infer from lockfiles. If none exist, default to npm for Node and pip for Python.

## Base Image Versions

**IMPORTANT**: Always detect the local runtime version to ensure dev/prod consistency. Do not rely on your training data for version numbers.

### Version Selection Priority

1. **Project config files** (highest priority)
2. **Local installed version** (run version command)
3. **Fallback to `references/base-images.md`** (only if above methods fail)

### Detect Local Version

Before choosing a base image, check for the runtime version in this order:

**Node.js:**
1. Check config files: `.nvmrc`, `.node-version`, `engines.node` in package.json, `volta.node` in package.json
2. If no config, run: `node --version`
3. Use major version for image tag (e.g., `v24.13.0` → `node:24-slim`)

**Python:**
1. Check config files: `.python-version`, `[tool.poetry.dependencies].python` in pyproject.toml, `python_requires` in Pipfile
2. If no config, run: `python --version` or `python3 --version`
3. Use major.minor for image tag (e.g., `3.12.1` → `python:3.12-slim`)

**Go:**
1. Check `go.mod` for `go` directive (e.g., `go 1.23`)
2. If no go.mod, run: `go version`
3. Use major.minor for image tag (e.g., `1.23.4` → `golang:1.23-alpine`)

**Rust:**
1. Check `rust-toolchain.toml` or `rust-toolchain` file, `rust-version` in Cargo.toml
2. If no config, run: `rustc --version`
3. Use major.minor for image tag (e.g., `1.83.0` → `rust:1.83-slim`)

**Ruby:**
1. Check config files: `.ruby-version`, `ruby` directive in Gemfile
2. If no config, run: `ruby --version`
3. Use major.minor for image tag (e.g., `3.3.0` → `ruby:3.3-slim`)

### Fallback Reference

If local detection fails, check `references/base-images.md` for current recommended versions.

### Image Tag Guidelines

- Use `slim` variants for smaller images (recommended for most apps)
- Use `alpine` for even smaller images (some compatibility tradeoffs)
- Always use specific major.minor tags, never `latest`

## Dockerfile Best Practices

### General Principles

- Use specific version tags, not `latest` (see Base Image Versions above)
- Minimize layers by combining RUN commands
- Order instructions from least to most frequently changing
- Use `.dockerignore` to exclude unnecessary files
- Run as non-root user in production
- Include EXPOSE for documentation

### Multi-stage Build Pattern

```dockerfile
# Build stage (verify current LTS version in references/base-images.md)
FROM node:24-slim AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:24-slim AS runner
WORKDIR /app
RUN addgroup -g 1001 -S appgroup && adduser -u 1001 -S appuser -G appgroup
COPY --from=builder --chown=appuser:appgroup /app/dist ./dist
COPY --from=builder --chown=appuser:appgroup /app/node_modules ./node_modules
USER appuser
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### Framework-Specific Considerations

**TanStack Start**: See `references/tanstack-start.md` for detailed instructions. Key points:
- Requires `"type": "module"` in package.json
- Requires `vite.config.ts` with `tanstackStart()` and `nitro()` plugins
- Build output goes to `.output/server/index.mjs`
- Use Node 24 slim with pnpm via corepack
- Include a `/health` route for health checks

**Next.js**: Use standalone output mode, copy `.next/standalone` and `.next/static`

**Vite/React**: Build static files, serve with nginx or a Node server

**Python/FastAPI**: Use slim images, install with `--no-cache-dir`

**Go**: Build static binary, use scratch or distroless for minimal image

## Output Format

After creating the Dockerfile, provide output in this order:

1. The complete Dockerfile
2. A `.dockerignore` file if one doesn't exist
3. Instructions to build and test locally:
   ```bash
   docker build -t myapp .
   docker run -p 3000:3000 myapp
   ```
4. Example `haloy.yaml` if one doesn't exist:
   ```yaml
   name: myapp
   server: user@server.example.com

   services:
     web:
       domains:
         - myapp.example.com
       port: 3000
       health_check: /health
   ```

## Reference Files

Read the appropriate reference file for detailed instructions:

- **Base Images**: `references/base-images.md` - Current recommended versions for all runtimes (check this first!)
- **TanStack Start**: `references/tanstack-start.md` - Complete guide including health checks, database setup, and haloy.yaml examples

## When to Create .dockerignore

Always check for an existing `.dockerignore`. If missing, create one appropriate for the project type. Common exclusions:

```
node_modules
.git
.env
.env.*
*.log
dist
.next
.output
.vinxi
__pycache__
*.pyc
.venv
target
```
