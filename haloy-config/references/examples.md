# Example Interactions

## With Configured Servers

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

## With Database Detection

**User**: "Create a haloy config for this project"

**Agent**:
1. Checks for Dockerfile - found
2. Checks `~/.config/haloy/client.yaml` - finds configured servers
3. Detects TanStack Start from package.json
4. Detects `@prisma/client` in dependencies
5. Checks `prisma/schema.prisma` - finds `provider = "postgresql"`

**Agent asks**:
> "I found these configured haloy servers. Which one would you like to deploy to?"
> - example.haloy.dev
> - prod.mycompany.com

**User**: "example.haloy.dev"

**Agent asks**:
> "What domain should this app be accessible at?"

**User**: "todo.example.com"

**Agent asks**:
> "Would you like to add any domain aliases?"

**User**: "no"

**Agent asks**:
> "I detected PostgreSQL database dependencies (Prisma). Would you like to add a self-hosted database service, or will you use an external provider (Supabase, Neon, RDS, etc.)?"
> - Add self-hosted database
> - Use external provider

**User**: "Add self-hosted database"

**Agent creates** `haloy.yaml`:
```yaml
server: "example.haloy.dev"
env:
  - name: POSTGRES_USER
    value: "postgres"
  - name: POSTGRES_PASSWORD
    value: "change-me-in-production"
  - name: POSTGRES_DB
    value: "todo_app"

targets:
  postgres:
    preset: database
    image:
      repository: postgres:17
    port: 5432
    volumes:
      - postgres-data:/var/lib/postgresql/data

  todo-app:
    domains:
      - domain: "todo.example.com"
    port: 3000
    env:
      - name: DATABASE_URL
        value: "postgres://postgres:change-me-in-production@postgres:5432/todo_app"
      - name: NODE_ENV
        value: "production"
```

**Agent provides next steps**:
> Your `haloy.yaml` has been created with a self-hosted PostgreSQL database.
>
> **Important**: Change `POSTGRES_PASSWORD` to a secure value before deploying to production.
>
> **DNS Setup**: Remember to configure DNS with your domain provider. Point `todo.example.com` to your haloy server's IP address.
>
> To deploy:
> 1. Validate the config: `haloy validate-config`
> 2. Deploy: `haloy deploy`

## With Secret Provider

**User**: "Create a haloy config, we use 1Password for secrets"

**Agent**:
1. Checks for Dockerfile - found
2. Checks `~/.config/haloy/client.yaml` - finds configured servers
3. Detects Express.js from package.json
4. Detects `pg` in dependencies (PostgreSQL)

**Agent asks**:
> "I found these configured haloy servers. Which one would you like to deploy to?"
> - prod.mycompany.com

**User**: "prod.mycompany.com"

**Agent asks**:
> "What domain should this app be accessible at?"

**User**: "api.mycompany.com"

**Agent asks**:
> "Would you like to add any domain aliases?"

**User**: "no"

**Agent asks**:
> "I detected PostgreSQL database dependencies. Would you like to add a self-hosted database service, or will you use an external provider?"
> - Add self-hosted database
> - Use external provider

**User**: "Use external provider"

**Agent creates** `haloy.yaml`:
```yaml
name: "my-api"
server: "prod.mycompany.com"
domains:
  - domain: "api.mycompany.com"
port: "3000"

secret_providers:
  onepassword:
    prod-db:
      vault: "Production"
      item: "Database Credentials"
    prod-secrets:
      vault: "Production"
      item: "API Secrets"

env:
  - name: "NODE_ENV"
    value: "production"
  - name: "DB_PASSWORD"
    from:
      secret: "onepassword:prod-db.password"
  - name: "DATABASE_URL"
    value: "postgres://app:${DB_PASSWORD}@db.mycompany.com:5432/myapi"
  - name: "JWT_SECRET"
    from:
      secret: "onepassword:prod-secrets.jwt-secret"
```

**Agent provides next steps**:
> Your `haloy.yaml` has been created with 1Password secret integration.
>
> **Prerequisites**: Make sure the 1Password CLI (`op`) is installed and authenticated, and that the referenced vault items exist.
>
> **DNS Setup**: Point `api.mycompany.com` to your haloy server's IP address.
>
> To deploy:
> 1. Validate: `haloy validate-config`
> 2. Check resolved secrets: `haloy validate-config --show-resolved-config`
> 3. Deploy: `haloy deploy`

## With Private Registry and Hooks

**User**: "Create a haloy config. We push images to GHCR and need to run migrations on deploy"

**Agent**:
1. Checks for Dockerfile - found
2. Checks `~/.config/haloy/client.yaml` - finds configured servers
3. Detects Next.js from package.json
4. Detects Prisma with PostgreSQL

**Agent asks**:
> "I found these configured haloy servers. Which one would you like to deploy to?"
> - prod.mycompany.com

**User**: "prod.mycompany.com"

**Agent asks**:
> "What domain should this app be accessible at?"

**User**: "app.mycompany.com"

**Agent asks**:
> "Would you like to add any domain aliases?"

**User**: "www.app.mycompany.com"

**Agent asks**:
> "I detected PostgreSQL database dependencies (Prisma). Would you like to add a self-hosted database service, or will you use an external provider?"

**User**: "External, we use Neon"

**Agent creates** `haloy.yaml`:
```yaml
name: "my-app"
server: "prod.mycompany.com"
domains:
  - domain: "app.mycompany.com"
    aliases:
      - "www.app.mycompany.com"
port: "3000"

image:
  repository: "ghcr.io/mycompany/my-app"
  tag: "latest"
  source: "registry"
  registry:
    username:
      from:
        env: "GHCR_USERNAME"
    password:
      from:
        env: "GHCR_TOKEN"

pre_deploy:
  - "npx prisma migrate deploy"
post_deploy:
  - "curl -s -X POST https://hooks.slack.com/services/XXX -d '{\"text\":\"my-app deployed\"}'"

env:
  - name: "DATABASE_URL"
    from:
      env: "NEON_DATABASE_URL"
  - name: "NODE_ENV"
    value: "production"
```

**Agent provides next steps**:
> Your `haloy.yaml` has been created with GHCR registry auth and deploy hooks.
>
> **Prerequisites**:
> - Set `GHCR_USERNAME` and `GHCR_TOKEN` locally (use a GitHub PAT with `read:packages` scope)
> - Set `NEON_DATABASE_URL` locally with your Neon connection string
>
> **DNS Setup**: Point `app.mycompany.com` and `www.app.mycompany.com` to your haloy server's IP address.
>
> The `pre_deploy` hook will run `prisma migrate deploy` before each deployment, and `post_deploy` will send a Slack notification after.
>
> To deploy:
> 1. Validate: `haloy validate-config`
> 2. Deploy: `haloy deploy`

## Without Configured Servers

**User**: "Create a haloy config"

**Agent**:
1. Checks for Dockerfile - found
2. Checks `~/.config/haloy/client.yaml` - no servers configured
3. Checks if haloy CLI is installed - not found

**Agent responds**:
> "No haloy servers configured. To deploy with haloy, you'll need to install the CLI and add a server first. See: https://haloy.dev/docs/client-installation"
>
> Once installed, run `haloy server add` to configure your server, then run this skill again.
