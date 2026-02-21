# Grafana Monitoring Stack

A self-hosted observability platform for homelab infrastructure, built with Grafana, InfluxDB, Telegraf, Loki, and Promtail. This project demonstrates how I used AI tooling to accelerate deployment and dashboard development without needing deep expertise in Flux query language or LogQL up front.

## The Problem

I had monitoring running on individual hosts — different tools, different interfaces, no unified view. Useful for spot-checking, but useless for trend analysis or catching slow-moving problems like gradual disk fill or creeping temperatures. Logs were scattered with no central aggregation.

The goal was a single pane of glass with persistent time-series data, log aggregation, actionable dashboards, and room to expand.

## AI-Assisted Workflow

I used Claude (via the official Grafana MCP server) to accelerate this build significantly. Rather than spending hours reading Flux query docs to get basic panels working, I could describe what I wanted in plain language and iterate from there.

Concrete examples of what this enabled:
- Generated working Flux queries for per-disk usage, I/O, and SMART temperature data
- Built the array growth rate and "days until full" prediction panels through conversation
- Configured the Grafana data source and dashboard JSON via MCP tools directly — no manual UI clicking for most of it
- Iterated on panel layout and thresholds faster than I could have manually

This isn't about replacing understanding — I reviewed and modified everything. It's about removing the friction of syntax and boilerplate so I could focus on what the monitoring should actually show.

## Stack

| Component | Role |
|-----------|------|
| **InfluxDB 2.7** | Time-series database for metrics |
| **Telegraf** | Metrics collection agent (golift fork for SMART support) |
| **Grafana** | Dashboards and visualization |
| **Loki** | Log aggregation |
| **Promtail** | Log shipping agent (migrating to Alloy) |

All deployed via Docker Compose on TrueNAS (atlas). Telegraf runs with host networking and elevated capabilities to access SMART data, disk metrics, and hardware sensors.

```
TrueNAS (atlas)
├── Telegraf ──────────────→ InfluxDB ──→ Grafana
│   (metrics: disk, SMART,                  ↑
│    CPU, net, UPS, docker)                 │
└── Promtail ──────────────→ Loki ─────────┘
    (logs: system, docker
     container logs)
```

External access via SWAG reverse proxy (`proxy` network).

## Dashboards

### Unraid Array Monitoring (15 panels)

Focused on storage health and capacity planning:

- Array usage gauge with current percentage
- Growth rate calculations (week / month / year)
- Days until full prediction based on observed growth rate
- Per-disk usage table (18 data disks)
- Per-disk I/O table (read/write)
- Drive temperature time-series graph (all drives)
- Average drive temperature stat
- Total array I/O
- UPS status, battery charge, load, and runtime remaining
- Estimated daily power cost

### Homelab System Health (10 panels)

Focused on system performance:

- System uptime
- CPU usage (gauge + time-series)
- Memory usage (gauge + time-series)
- Network traffic in/out (bidirectional graph)
- System load average (1/5/15 min)
- Cache disk usage
- Parity check status

## Key Configuration Notes

**Telegraf** uses the `golift/telegraf` image which includes `smartmontools` for SMART data collection. Runs with host networking and `SYS_ADMIN` / `SYS_PTRACE` capabilities. Host filesystem mounted at `/hostfs`.

**InfluxDB** setup:
- Organization: `homelab`
- Bucket: `unraid`
- Flux query language (v2 — not InfluxQL)

**Loki + Promtail** collect system logs and Docker container logs. Promtail scrapes `/var/log` and the Docker socket for container log discovery.

**Grafana** has the Netdata datasource plugin pre-installed (`GF_PLUGINS_PREINSTALL`) alongside InfluxDB and Loki datasources.

**Networks:** Two Docker networks — `monitoring` (internal stack communication) and `proxy` (external, shared with SWAG reverse proxy).

## Structure

```
grafana-monitoring-stack/
├── compose/
│   └── compose.yaml          # Full stack: InfluxDB, Grafana, Telegraf, Loki, Promtail
├── telegraf/
│   └── telegraf.conf         # Metrics collection config
├── dashboards/
│   ├── unraid.json           # Unraid Array Monitoring dashboard export
│   └── system-health.json    # Homelab System Health dashboard export
├── .env.example              # Credential template
└── README.md
```

## Environment Variables

Copy `.env.example` to `.env` and fill in before deploying:

```env
INFLUXDB_ADMIN_USER=
INFLUXDB_ADMIN_PASSWORD=
INFLUXDB_ORG=homelab
INFLUXDB_BUCKET=unraid
INFLUXDB_ADMIN_TOKEN=

GRAFANA_ADMIN_USER=
GRAFANA_ADMIN_PASSWORD=
GRAFANA_ROOT_URL=
```

## Why This Stack

I considered Prometheus + Grafana (more moving parts, pull-based model less suited to my setup) and Netdata standalone (less flexible for custom dashboards and long-term retention). InfluxDB fit better for a single-user homelab focused on trend analysis — simpler to operate, excellent Grafana integration, and Telegraf handles collection with minimal config. Adding Loki gave log aggregation without introducing another query language or UI.

---

*Deployed: February 2026 | Running on TrueNAS Scale*
