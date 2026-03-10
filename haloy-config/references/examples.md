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
    image: "postgres:18"
    port: 5432
    volumes:
      - postgres-data:/var/lib/postgresql

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

## With a Published Image

**User**: "Create a haloy config for this service. The image is already published to GHCR."

**Agent**:
1. Checks for Dockerfile - not required because the user already has a published image
2. Checks `~/.config/haloy/client.yaml` - finds configured servers
3. Detects a Node.js service from `package.json`
4. Infers app name "event-worker"
5. Notes this should use explicit image configuration instead of Haloy's default local build flow

**Agent asks**:
> "I found these configured haloy servers. Which one would you like to deploy to?"
> - example.haloy.dev
> - prod.mycompany.com

**User**: "prod.mycompany.com"

**Agent asks**:
> "What published image should Haloy deploy? For example: `ghcr.io/acme/event-worker:v1.2.3`"

**User**: "ghcr.io/acme/event-worker:v1.2.3"

**Agent asks**:
> "Does this service need a public domain, or is it an internal worker?"

**User**: "Internal worker"

**Agent creates** `haloy.yaml`:
```yaml
name: "event-worker"
server: "prod.mycompany.com"
image: "ghcr.io/acme/event-worker:v1.2.3"
```

**Agent provides next steps**:
> Your `haloy.yaml` has been created to deploy the published GHCR image directly.
>
> To deploy:
> 1. Validate the config: `haloy validate-config`
> 2. Deploy: `haloy deploy`

## With Multiple Images and `image_key`

**User**: "Create a haloy config for this app. I need a web target and a worker target with different images."

**Agent**:
1. Checks for Dockerfile - found
2. Checks `~/.config/haloy/client.yaml` - finds configured servers
3. Detects a monorepo layout where web and worker should not share the same image
4. Chooses root `images` plus target `image_key` instead of duplicating image blocks per target

**Agent asks**:
> "I found these configured haloy servers. Which one would you like to deploy to?"
> - example.haloy.dev
> - prod.mycompany.com

**User**: "example.haloy.dev"

**Agent asks**:
> "What domain should the web target use?"

**User**: "app.example.com"

**Agent creates** `haloy.yaml`:
```yaml
name: "acme-app"
server: "example.haloy.dev"
images:
  web:
    repository: "acme-web"
    tag: "latest"
    build_config:
      context: "."
      dockerfile: "./apps/web/Dockerfile"
  worker:
    repository: "acme-worker"
    tag: "latest"
    build_config:
      context: "."
      dockerfile: "./apps/worker/Dockerfile"

targets:
  production:
    image_key: "web"
    domains:
      - domain: "app.example.com"
    port: 3000

  jobs:
    image_key: "worker"
```

**Agent provides next steps**:
> Your `haloy.yaml` has been created with named images so each target can resolve the correct image without duplicating configuration.
>
> **DNS Setup**: Point `app.example.com` to your haloy server.
>
> To deploy:
> 1. Validate the config: `haloy validate-config`
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
