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
2. Detects configured haloy servers from `~/.config/haloy/client.yaml`
3. Detects your project type and framework
4. Infers defaults for app name, port, and health check path
5. Asks you to select a server (from configured list) and domain
6. Creates a `haloy.yaml` configuration file
7. Provides next steps for deployment

If no servers are configured, the skill will guide you to install the haloy CLI first.

## Example

**You**: "Create a haloy config"

**Agent**: Detects your Next.js project, finds configured servers, then asks:

> I found these configured haloy servers. Which one would you like to deploy to?
> - .haloy.dev
> - prod.mycompany.com

**You**: "example.haloy.dev"

> What domain should this app be accessible at?

**You**: "myapp.example.com"

> Would you like to add any domain aliases? (e.g., www.myapp.example.com)

**You**: "yes, www.myapp.example.com"

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

**Agent reminds**:
> Remember to configure DNS with your domain provider. Point your domains to your haloy server's IP address.

## Related Skills

- **dockerize** - Create a Dockerfile for your project (run this first if you don't have one)

## Documentation

- [Haloy Configuration Reference](https://haloy.dev/docs/configuration-reference)
- [Client Installation](https://haloy.dev/docs/client-installation)
- [Server Installation](https://haloy.dev/docs/server-installation)

## License

MIT
