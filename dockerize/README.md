# dockerize

Create production-ready Dockerfiles for deployment with [haloy](https://haloy.dev).

## Installation

```bash
npx skills add haloydev/agent-skills
```

## Usage

Trigger the skill by asking your AI agent to:

- "Dockerize this app"
- "Create a Dockerfile"
- "Containerize my application"
- "Make this deployable with haloy"
- "Prepare this for production"

## What It Does

1. Detects your project type (TanStack Start, Next.js, Node.js, Python, Go, Rust, etc.)
2. Checks for a health endpoint and offers to create one if missing
3. Analyzes build process and runtime requirements
4. Creates an optimized multi-stage Dockerfile with HEALTHCHECK
5. Generates a `.dockerignore` if needed
6. Advises using `/haloy-config` skill or docs for `haloy.yaml` setup

## Health Checks

The skill will ask if you want to add a `/health` endpoint if one doesn't exist. Health checks are important for haloy because they enable:

- **Zero-downtime deployments** - New containers must be healthy before receiving traffic
- **Auto-recovery** - Unhealthy containers are automatically restarted
- **Fast failure detection** - Broken deployments fail quickly instead of silently

## Supported Frameworks

- **TanStack Start** - Full support with Nitro server, health checks, and database patterns
- **Next.js** - Standalone output mode
- **Vite/React** - Static builds
- **Python/FastAPI** - Slim images
- **Go** - Static binaries
- And more...

## Examples

### TanStack Start

```
User: Dockerize this TanStack Start app

Agent: I'll create an optimized Dockerfile for your TanStack Start application...

[Creates Dockerfile with pnpm, Nitro output, health check]
[Creates /health route if missing]
[Creates .dockerignore]
[Suggests running /haloy-config for configuration]
```

### Next.js

```
User: Dockerize this Next.js app

Agent: I'll create an optimized Dockerfile for your Next.js application.
       Your app doesn't have a health endpoint. Would you like me to create one?

User: Yes

Agent: [Creates /health route]
       [Creates Dockerfile with standalone output, HEALTHCHECK, non-root user]
       [Creates .dockerignore]
       [Suggests running /haloy-config for configuration]
```

## Note on Docker Compose

This skill creates single Dockerfiles for haloy deployment. Haloy handles orchestration through its own configuration, so docker-compose is not needed. If you need multiple services, each should have its own Dockerfile and be defined separately in your `haloy.yaml`.

## License

MIT
