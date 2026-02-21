# Grafana Monitoring Stack

A self-hosted observability platform for homelab infrastructure, built with Grafana, InfluxDB, and Telegraf. This project demonstrates how I used AI tooling to accelerate deployment and dashboard development without needing deep expertise in Flux query language up front.

## The Problem

I had monitoring running on individual hosts — different tools, different interfaces, no unified view. Useful for spot-checking, but useless for trend analysis or catching slow-moving problems like gradual disk fill or creeping temperatures.

The goal was a single pane of glass with persistent time-series data, actionable dashboards, and room to expand.

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
| **InfluxDB 2.7** | Time-series database |
| **Telegraf** | Metrics collection agent (runs on Unraid) |
| **Grafana** | Dashboards and visualization |

Deployed via Docker Compose on TrueNAS (atlas). Telegraf runs as a privileged container on Unraid to access SMART data and disk metrics.

```
Unraid
└── Telegraf ──→ InfluxDB (atlas) ──→ Grafana (atlas)
```

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

**Telegraf requires:**
- Privileged container mode (for SMART data via `smartctl`)
- Volume mounts: `/`, `/proc`, `/sys`, `/var/run/docker.sock`
- `HOST_PROC`, `HOST_SYS`, `HOST_MOUNT_PREFIX` environment variables

**InfluxDB setup:**
- Organization: `homelab`
- Bucket: `unraid`
- Flux query language (v2 — not InfluxQL)

**Grafana MCP integration:**
- Configured via Claude Desktop
- Enabled AI-assisted dashboard creation and editing via conversation
- Used for majority of initial panel and query development

## Structure

```
grafana-monitoring-stack/
├── compose/
│   └── compose.yaml          # InfluxDB + Grafana stack
├── telegraf/
│   └── telegraf.conf         # Unraid metrics collection config
├── dashboards/
│   ├── unraid.json           # Unraid Array Monitoring dashboard export
│   └── system-health.json    # Homelab System Health dashboard export
└── README.md
```

## Why This Stack

I considered Prometheus + Grafana (more moving parts, pull-based model less suited to my setup) and Netdata (less flexible for custom dashboards and long-term retention). InfluxDB fit better for a single-user homelab focused on trend analysis — simpler to operate, excellent Grafana integration, and Telegraf handles collection with minimal config.

---

*Deployed: February 2026 | Running on TrueNAS Scale + Unraid*
