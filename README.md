# Home Server Docker Compose Collection

A collection of Docker Compose stacks used to run a personal home server. Each top-level directory is an independent, self-contained service that can be started and stopped without affecting the others.

## Prerequisites

- Docker Engine 24 or later
- Docker Compose v2 (the `docker compose` plugin)
- A Linux host with enough disk space for the volumes each service creates

## Repository Structure

Each directory contains its own `docker-compose.yml` (and, where relevant, a `docker-compose.dev.yml` variant) plus a `.env.example` file documenting the environment variables that service expects.

| Directory | Service | Purpose |
|---|---|---|
| `dashdot` | Dashdot | System resource dashboard |
| `gitea` | Gitea + PostgreSQL | Lightweight self-hosted Git service |
| `gitlab-ce` | GitLab CE + GitLab Runner | Full-featured self-hosted Git platform and CI runner |
| `grafana` | Grafana, Prometheus, Loki, Tempo, Mimir, Pushgateway | Observability stack (metrics, logs, traces) |
| `homarr` | Homarr | Dashboard/homepage for the other services |
| `jeager` | Jaeger, OpenTelemetry Collector | Distributed tracing, feeds the Grafana stack |
| `jenkins` | Jenkins, plus a separate build agent under `jenkins/agent` | CI/CD server and Docker-capable runner |
| `keycloak` | Keycloak + PostgreSQL | Identity and access management |
| `linkwarden` | Linkwarden, PostgreSQL, Meilisearch | Bookmark manager and archiver |
| `metabase` | Metabase | Business intelligence and analytics |
| `minio` | MinIO | S3-compatible object storage |
| `mongodb` | MongoDB | Standalone database instance |
| `monitoring` | Prometheus, Grafana, node-exporter, cAdvisor | Host and container metrics stack |
| `mysql` | MySQL | Standalone database instance |
| `n8n` | n8n + PostgreSQL | Workflow automation |
| `nextcloud` | Nextcloud + PostgreSQL | File storage and sync |
| `ollama` | Ollama | Local LLM runtime |
| `portainer` | Portainer CE | Docker management UI |
| `postgres` | PostgreSQL + pgAdmin | Standalone database instance with admin UI |
| `vaultwarden` | Vaultwarden | Bitwarden-compatible password manager |
| `wordpress` | WordPress + MySQL | Blog/CMS |

## Getting Started

Each service is started independently from within its own directory.

1. Copy the example environment file and fill in real values:

   ```bash
   cd <service-directory>
   cp .env.example .env
   ```

2. Edit `.env` and replace every `CHANGE_ME` placeholder with a strong, unique value.

3. Start the stack:

   ```bash
   docker compose up -d
   ```

4. To stop a stack:

   ```bash
   docker compose down
   ```

Some services (for example `mysql`, `mongodb`, and `postgres`) also provide a `docker-compose.dev.yml` for local development, paired with a `.env.dev.example` file:

```bash
cp .env.dev.example .env.dev
docker compose -f docker-compose.dev.yml up -d
```

### Service Dependencies

The `jeager` stack attaches to the external Docker network created by the `grafana` stack (`grafana_monitoring`). Start `grafana` before starting `jeager`:

```bash
cd grafana && docker compose up -d
cd ../jeager && docker compose up -d
```

## Environment Variables and Secrets

No credentials are committed to this repository. Every value that would normally be a password, token, or key is read from an environment variable and supplied through a local `.env` file, which is excluded from version control by `.gitignore`.

Each service directory includes a `.env.example` file listing the variables it needs, with placeholder values. Copy it to `.env`, fill in real values, and never commit the resulting file.

Runtime data directories created by containers (database volumes, Jenkins home, Vaultwarden data, and similar) are also excluded by `.gitignore` and never tracked.

## Security Notes

- Change every default or placeholder credential before exposing a service beyond your local network.
- Several services (Keycloak, Grafana, Gitea) ship with weak defaults in their example configuration purely for local bring-up. Treat these as unsafe for anything but a first run behind a closed network.
- None of these stacks configure TLS termination on their own. Put a reverse proxy (Caddy, Traefik, or Nginx) in front of any service reachable from outside your local network.
- Review the `docker-compose.yml` for each service before deploying it publicly. Several bind-mount the Docker socket (`/var/run/docker.sock`), which grants the container effective root access to the host.

## License

This repository documents a personal infrastructure setup and is provided as-is, with no warranty of any kind.
