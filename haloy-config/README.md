# haloy-config

Create `haloy.yaml` configuration files for deploying applications with [haloy](https://haloy.dev).

## Installation

```bash
npx skills add haloydev/agent-skills
```

## Usage

Trigger phrases:
- "Create haloy config"
- "Add haloy.yaml"
- "Configure for haloy"
- "Set up haloy deployment"
- "Prepare for haloy"

## What It Does

1. Checks for a Dockerfile (suggests `dockerize` skill if missing)
2. Detects your project type and framework
3. Infers defaults for app name, port, and health check path
4. Asks for your haloy server URL and domain
5. Creates a `haloy.yaml` configuration file
6. Provides next steps for deployment

## Example

**You**: "Create a haloy config"

**Agent**: Detects your Next.js project, finds the health endpoint, then asks:

> What is your haloy server URL?

**You**: "api.myserver.com"

> What domain should this app be accessible at?

**You**: "myapp.example.com"

**Agent creates** `haloy.yaml`:

```yaml
name: "my-nextjs-app"
server: api.myserver.com
domains:
  - domain: "myapp.example.com"
port: "3000"
health_check_path: "/api/health"
```

## Related Skills

- **dockerize** - Create a Dockerfile for your project (run this first if you don't have one)

## Documentation

- [Haloy Configuration Reference](https://haloy.dev/docs/configuration-reference)
- [Client Installation](https://haloy.dev/docs/client-installation)
- [Server Installation](https://haloy.dev/docs/server-installation)

## License

MIT
