# monitoring-stack-prometheus-grafana

A self-contained Prometheus + Grafana + node-exporter stack, spun up with a single `docker-compose up` — includes alerting rules and a pre-provisioned dashboard, not just "here's how to install Prometheus."

## What this demonstrates

- Full observability stack wired together: metrics collection (node-exporter) → storage/querying (Prometheus) → visualization (Grafana)
- Grafana datasource and dashboard **provisioned automatically** on container start — no manual "add a datasource, import a dashboard" clicking required
- Alerting rules written in PromQL for the failure modes that actually matter first: sustained high CPU, high memory pressure, and target-down detection
- A dashboard with real PromQL queries (not screenshots) checked into the repo, so it's reviewable in a PR like any other config

## Architecture

```
┌────────────────┐      scrapes      ┌────────────┐      queried by      ┌─────────┐
│  node-exporter  │ ◀──────────────── │ Prometheus │ ◀──────────────────  │ Grafana │
│  (host metrics)  │                  │  + alerts   │                      │  :3001  │
└────────────────┘                   └────────────┘                      └─────────┘
       :9100                              :9090
```

## Prerequisites

- Docker + Docker Compose

## Usage

```bash
docker compose up -d
```

- Prometheus: [http://localhost:9090](http://localhost:9090) — check **Status → Targets** to confirm both `prometheus` and `node-exporter` are `UP`
- Grafana: [http://localhost:3001](http://localhost:3001) — login `admin` / `admin` (you'll be prompted to change it). The "Node Exporter Overview" dashboard is already there, no import step needed.
- Alert rules: visible under **Alerts** in Prometheus once `node_exporter` has been scraped for a few minutes.

## Design decisions

**Why provision the dashboard via a JSON file instead of clicking through the Grafana UI?** A dashboard built by clicking is invisible to version control — nobody can see what changed or review it in a PR. Provisioning it as JSON means dashboard changes go through the same review process as any other infrastructure change.

**Why these three specific alerts first?** CPU, memory, and instance-down cover the failure modes that actually page someone at 3am. It's deliberately not a wall of 40 alerts on day one — alert fatigue from over-broad thresholds is what causes real alerts to get ignored. I'd add service-specific alerts (error rate, latency) once there's an actual application exporting its own metrics, which pairs naturally with the `cicd-pipeline-demo` app in this same portfolio.

**What I'd add before this runs against real production hosts:** Alertmanager for routing/deduplication/silencing (right now alerts fire in Prometheus but nothing sends them anywhere), TLS + real auth in front of both UIs, and `node-exporter` running as a host-level systemd service or DaemonSet rather than a single container, since it needs to see the actual host's metrics, not a container's.
