---
name: docker-compose-scaffold
description: Multi-service local development environment setup with docker-compose, service configuration, networking, and developer experience tooling
---

## What This Skill Does

Defines the workflow for setting up a local development environment using Docker Compose. Creates a complete, reproducible local environment with all application services, databases, caches, and supporting infrastructure.

## When To Use

Use this when:
- Setting up a new project's local development environment
- Adding a new dependency service (database, cache, message queue) to an existing project
- Creating a complete local stack for integration testing
- Onboarding developers who need to run the full stack locally

Do NOT use this when:
- Deploying to production (use devops-engineer + terraform-workflow)
- Setting up CI pipelines (use ci-ops)
- The project is a standalone CLI tool or library with no service dependencies

## Workflow

### Phase 1: Inventory Services
**Agent**: planner

1. Identify all services the application needs to run locally:
   - Application service(s) (Go, Node.js, Java)
   - Database(s) (PostgreSQL, MySQL, MongoDB)
   - Cache (Redis, Memcached)
   - Message queue (RabbitMQ, Kafka)
   - Search (Elasticsearch, Meilisearch)
   - Mail (Mailhog, Mailpit for local email testing)
   - Observability (Grafana, Prometheus, Jaeger)
2. Identify which services need persistent volumes
3. Identify which services need health checks
4. Identify port mappings needed for local development

**Gate**: Service inventory must be confirmed before composing.

### Phase 2: Write docker-compose.yml
**Agent**: devops-engineer

1. Create `docker-compose.yml` (or `docker-compose.dev.yml` if a production compose exists)
2. Configure each service with:
   - **Image**: Use specific version tags, never `latest`
   - **Ports**: Map to localhost for development access
   - **Environment**: Configuration via environment variables
   - **Volumes**: Named volumes for data persistence, bind mounts for live code reloading
   - **Health checks**: For all services that support them
   - **Networks**: Default bridge network unless services need isolation
   - **Depends_on**: With `condition: service_healthy` for ordering
3. Create `.env.example` with all required environment variables (no real secrets)

**Important**: Research current image tags for all services before writing. Do not use versions from memory.

### Phase 3: Application Service Configuration
**Agent**: builder, ts-builder, or java-builder (depending on stack)

1. Create/update application Dockerfile for development (with hot-reload):
   - Mount source code via volume (not copy)
   - Install dev dependencies
   - Use a file watcher for automatic restart (nodemon, air, spring-boot-devtools)
2. Create/update application config to read from environment variables
3. Create seed data scripts (if applicable):
   - Database schema initialization
   - Sample data for development
4. Create a `scripts/dev-setup.sh` (or Makefile targets) for first-time setup

### Phase 4: Networking and Security
**Agent**: devops-engineer

1. Verify no port conflicts with common local services
2. Use `.env` for all configurable values (ports, credentials, database names)
3. Use non-default passwords even for local development (avoid `postgres`/`postgres`)
4. Verify services only expose ports needed for development (not management ports)

### Phase 5: Developer Experience
**Agent**: shell-scripter

Create helper scripts/Makefile targets:
1. `make up` / `make down` — start and stop the stack
2. `make logs` — tail all service logs
3. `make reset` — destroy volumes and recreate from scratch
4. `make seed` — load sample data
5. `make ps` — show service status with health
6. `make shell-<service>` — open a shell in a running container

### Phase 6: Documentation
**Agent**: docs-writer

Update `README.md` with:
1. Prerequisites (Docker, Docker Compose version)
2. Quick-start: `make up` or `docker compose up`
3. Service URLs and ports (application, database admin, monitoring)
4. Default credentials for local services
5. Troubleshooting common issues (port conflicts, volume permissions)

### Phase 7: Verify
**Agent**: devops-engineer

1. Run `docker compose config` to validate the compose file
2. Run `docker compose up` and verify all services start and pass health checks
3. Verify the application can connect to all dependencies
4. Verify data persistence across `docker compose down` / `docker compose up` cycles
5. Verify `make reset` produces a clean, working environment

**Gate**: Full stack starts and works end-to-end.

## Principles

- **Pin all image versions**: `postgres:16-alpine`, not `postgres:latest`. Reproducibility is non-negotiable.
- **Health checks on everything**: Use `healthcheck` directives so `depends_on` actually waits for readiness.
- **Environment variables for configuration**: Never hardcode connection strings. Always `.env` file.
- **Named volumes for persistence**: Data survives `docker compose down`. Only `make reset` destroys data.
- **Fast startup**: Minimize build steps. Use pre-built images where possible. Cache layers aggressively.
- **Match production**: Use the same database engine and version locally as in production. Never PostgreSQL locally and MySQL in production.
