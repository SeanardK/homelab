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
| `ai-agents` | Empty Python+Node sandbox container (`agent-sandbox`), more to come | AI agent framework exploration sandbox, wired to Claude via Anthropic API |
| `cloudflared` | Cloudflare Tunnel (`network_mode: host`) | Routes the public `*.seanard.my.id` domains to services on this host without opening ports |
| `dashdot` | Dashdot | System resource dashboard |
| `garage` | Garage (single node) | Lightweight S3-compatible object storage; secrets live in `.env`, not in `garage.toml` |
| `gitea` | Gitea + PostgreSQL | Self-hosted Git service; its built-in container registry (Packages) hosts Jenkins-built images |
| `gitlab-ce` | GitLab CE + GitLab Runner | Full-featured self-hosted Git platform and CI runner |
| `grafana` | Grafana, Prometheus, Loki, Tempo, Mimir, Pushgateway, Alloy, node-exporter, cAdvisor | Full observability stack: metrics (app + host + container), logs, traces |
| `homarr` | Homarr | Dashboard/homepage for the other services |
| `jeager` | Jaeger, OpenTelemetry Collector | Distributed tracing, feeds the Grafana stack |
| `jenkins` | Jenkins, plus a separate build agent under `jenkins/agent` | CI/CD server and Docker-capable runner |
| `keycloak` | Keycloak + PostgreSQL | Identity and access management |
| `linkwarden` | Linkwarden, PostgreSQL, Meilisearch | Bookmark manager and archiver |
| `metabase` | Metabase | Business intelligence and analytics |
| `minio` | MinIO | S3-compatible object storage |
| `mongodb` | MongoDB (dev + prod, independent instances) | Standalone database instances |
| `mysql` | MySQL (dev + prod, independent instances) | Standalone database instances; `mysql_prod` also backs `wordpress` |
| `n8n` | n8n + PostgreSQL | Workflow automation |
| `nextcloud` | Nextcloud + PostgreSQL | File storage and sync |
| `ollama` | Ollama | Local LLM runtime |
| `portainer` | Portainer CE | Docker management UI |
| `postgres` | PostgreSQL + pgAdmin (dev + prod, independent instances) | Standalone database instances with admin UI |
| `redis` | Redis (dev + prod, independent instances) | Standalone cache/key-value store instances, password-protected with AOF persistence |
| `security-lab` | Kali Linux (headless) + Juice Shop, DVWA, WebGoat/WebWolf | Isolated ethical-hacking practice environment — attacker box + vulnerable targets |
| `sonarqube` | SonarQube + PostgreSQL | Code quality & security static analysis, integrates with Jenkins pipelines |
| `traefik` | Traefik | Reverse proxy — future portfolio projects opt in via docker labels instead of raw port mapping |
| `uptime-kuma` | Uptime Kuma | Uptime/status monitoring, can double as a public status page |
| `vaultwarden` | Vaultwarden | Bitwarden-compatible password manager |
| `wordpress` | WordPress | Blog/CMS. Connects to the existing `mysql_prod` database (see `mysql/`) instead of running its own MySQL |

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

Some services (for example `mysql`, `mongodb`, `postgres`, and `redis`) also provide a `docker-compose.dev.yml` for local development, paired with a `.env.dev.example` file:

```bash
cp .env.dev.example .env.dev
docker compose --env-file .env.dev -f docker-compose.dev.yml up -d
```

⚠️ **Always pass `--env-file .env.dev` explicitly when using the `.dev.yml` file.** Compose resolves `${VAR}` interpolation (inside `command:`/`environment:` blocks) from the project directory's default `.env` file regardless of which `-f` compose file you pass — `env_file:` inside the compose file only injects runtime env vars into the container, it does *not* control interpolation. Since prod's `.env` and dev's `.env.dev` sit in the same directory, running `docker compose -f docker-compose.dev.yml up -d` without `--env-file .env.dev` will silently bake the **prod** secret into the dev container instead of the dev one.

### Service Dependencies

The `jeager` stack attaches to the external Docker network created by the `grafana` stack (`grafana_monitoring`). Start `grafana` before starting `jeager`:

```bash
cd grafana && docker compose up -d
cd ../jeager && docker compose up -d
```

The `wordpress` stack attaches to the external Docker network created by the
`mysql` stack (`mysql_prod_network`) and uses the existing `mysql_prod`
container as its database — it does not run its own MySQL. Start `mysql`
first, create the WordPress database/user inside it once (see below), then
start `wordpress`:

```bash
cd mysql && docker compose up -d

# one-time: create the wordpress database + user inside mysql_prod,
# using the same values you put in wordpress/.env
docker exec -it mysql_prod mysql -uroot -p"$(grep MYSQL_ROOT_PASSWORD ../mysql/.env | cut -d= -f2)" -e \
  "CREATE DATABASE IF NOT EXISTS wordpress_db; \
   CREATE USER IF NOT EXISTS 'wp_user'@'%' IDENTIFIED BY 'REPLACE_WITH_WORDPRESS_DB_PASSWORD'; \
   GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wp_user'@'%'; \
   FLUSH PRIVILEGES;"

cd ../wordpress && docker compose -f seanard-docker-compose.yml up -d
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
