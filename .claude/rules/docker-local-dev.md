# Rule: Docker & Docker Compose for Local Development Infrastructure

Use this rule when generating or editing `Dockerfile` or `compose.yaml` files for **local development servers and tooling** (databases, caches, proxies, mock services, admin UIs, etc.).

---

## Compose File Conventions

- Use `compose.yaml` as the filename (Compose V2 canonical name)
- Do **not** include a `version:` key — it is obsolete in Compose V2
- Use `docker compose` (not `docker-compose`)

## Images

- Use official images from Docker Hub
- Pin to **minor versions** (e.g. `postgres:16`, `redis:7`, `nginx:1.26`) — not `latest`, not exact patch versions
- Prefer images that are well-maintained and widely used; favour `-alpine` variants only if you have a reason

## Healthchecks & Startup Order

Always define a `healthcheck` on services that other services depend on, and always use `condition: service_healthy` in `depends_on`. This prevents race-condition failures on first start.

```yaml
services:
  db:
    image: postgres:16
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-postgres}"]
      interval: 5s
      timeout: 5s
      retries: 5

  app:
    depends_on:
      db:
        condition: service_healthy
```

## Volumes

- Use **named volumes** for persistent data (databases, etc.) — avoids permissions issues with bind mounts
- Use **bind mounts** for config files you want to edit live (e.g. nginx config, seed SQL scripts)

```yaml
volumes:
  db_data:

services:
  db:
    volumes:
      - db_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
```

## Ports

- Always expose ports explicitly to localhost so local tools (psql, Redis Insight, etc.) can connect directly
- Document what each port is in a comment if it's not obvious

## Restart Policy

Use `restart: unless-stopped` on all persistent services so they survive Docker/machine restarts without manual intervention.

## Profiles

Use `profiles:` to keep optional tooling (admin UIs, mock services, debug tools) out of the default `docker compose up`. Developers opt in explicitly.

```yaml
services:
  pgadmin:
    image: dpage/pgadmin4:8
    profiles: [tools]
```

Start optional services with: `docker compose --profile tools up`

## What to Skip for Local Dev

These are **not** needed for local development infrastructure and add friction:

- Non-root users (causes bind mount permission headaches)
- Multi-stage builds (slower, no benefit locally)
- Resource limits
- Image size optimisation
- `.dockerignore` for infra-only containers

---

## Example: Minimal Postgres + Redis Stack

```yaml
services:
  db:
    image: postgres:16
    restart: unless-stopped
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: app
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U dev"]
      interval: 5s
      timeout: 5s
      retries: 5

  cache:
    image: redis:7
    restart: unless-stopped
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  db_data:
```
