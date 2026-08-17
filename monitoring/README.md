# Prometheus + Grafana (docker-compose)

Start the monitoring stack:

```bash
cd docker/monitoring
docker compose up -d
```

Access:
- Grafana: http://localhost:3000  (admin / admin)
- Prometheus: http://localhost:9090

- cAdvisor (container metrics): http://localhost:8080

Provisioned Grafana dashboards:
- Node Exporter - Basic: shown on startup
- Docker / Containers (cAdvisor): shown on startup

Files created:
- docker/monitoring/docker-compose.yml
- docker/monitoring/prometheus/prometheus.yml
- docker/monitoring/grafana/provisioning/datasources/prometheus.yml
