# Monitoring Stack: Prometheus + Grafana + Alertmanager

A ready-to-run monitoring stack for Docker hosts, built with Docker Compose.
It collects host and container metrics, visualises them in Grafana and fires
alerts for common failures (container down, low disk, high CPU/memory).

## Architecture

```
 node-exporter (host metrics) ─┐
                               ├─► Prometheus ─► Alertmanager (alerts)
 cAdvisor (container metrics) ─┘        │
                                        ▼
                                     Grafana (dashboards)
```

| Component | Purpose | Local URL |
|---|---|---|
| Prometheus | Scrapes and stores metrics, evaluates alert rules | http://localhost:9090 |
| Alertmanager | Groups and routes alerts | http://localhost:9093 |
| Grafana | Dashboards and visualisation | http://localhost:3000 |
| node-exporter | Host CPU, memory, disk, network metrics | internal only |
| cAdvisor | Per-container CPU, memory, network metrics | internal only |

## Security notes

- All UIs are bound to `127.0.0.1`; nothing is exposed to the network.
- `node-exporter` and `cAdvisor` publish no host ports.
- Grafana credentials come from a git-ignored `.env` file.
- For remote access, place a reverse proxy behind a VPN instead of opening ports.

## Quick start

```bash
git clone <your-repo-url>
cd monitoring-stack
cp .env.example .env        # then edit .env and set a strong password
docker compose up -d
docker compose ps
```

Open Grafana at http://localhost:3000 and log in with the credentials from `.env`.
The Prometheus data source is provisioned automatically.

## Dashboards

In Grafana go to **Dashboards > New > Import** and import:

- **1860**: Node Exporter Full (host metrics)
- **14282**: cAdvisor / Docker container metrics

Select the `Prometheus` data source when asked.

## Alerts included

| Alert | Condition | Severity |
|---|---|---|
| TargetDown | A scrape target is unreachable for 1 min | critical |
| ContainerDown | A container has not been seen for 60 s | critical |
| DiskSpaceLow | Less than 15% disk free for 5 min | warning |
| HighMemoryUsage | Memory above 85% for 5 min | warning |
| HighCPUUsage | CPU above 85% for 10 min | warning |

View them at http://localhost:9090/alerts and http://localhost:9093.

## Test an alert

```bash
docker stop cadvisor          # TargetDown / ContainerDown should fire after ~1-2 min
docker start cadvisor
```

## Notes for macOS / Windows (Docker Desktop)

node-exporter and cAdvisor report metrics of Docker Desktop's Linux VM, not the
host OS, and some container labels may be limited. For fully accurate results
run the stack on a Linux host.

## Clean up

```bash
docker compose down        # keep data
docker compose down -v     # also delete stored metrics and dashboards
```

## Screenshots

_Add screenshots of the Grafana dashboards and the Prometheus alerts page here._
