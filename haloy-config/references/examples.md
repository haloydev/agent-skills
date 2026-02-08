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
