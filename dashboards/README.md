# Dashboard JSON Exports

## How to export from your Grafana instance

1. Open the dashboard
2. Click the **Share** icon (top bar) → **Export** → **Save to file**
3. Replace the stubs in this directory with the exported JSON files

The JSON files in this directory are panel manifests only (not importable).
Full exportable JSON requires a running Grafana instance with live data.

## Datasource requirements

When importing, Grafana will prompt you to map datasource UIDs.
You will need:
- **InfluxDB** datasource pointing to your InfluxDB v2 instance (org: `homelab`, bucket: `unraid` or `claudebox`)  
- **Loki** datasource pointing to your Loki instance

## Dashboard summaries

### claudebox-system.json — Claudebox System Dashboard (25 panels)
Host: Debian (claudebox) | Datasources: InfluxDB + Loki

Sections:
- Stat row: uptime, RAM total/used%, processes, net in/out, root disk free
- CPU usage time-series (user/system/iowait/nice)
- System load (1/5/15 min)
- Memory (used/cached/buffered stacked area)
- Disk space per mount (bar gauge)
- Network traffic bidirectional graph
- Disk I/O bytes and IOPS
- Temperature sensors (CPU, NVMe, GPU, DDR5 RAM)
- Log error/warning rate bar chart (Loki)
- Syslog filtered live panel (Loki)
- GPU utilization % (AMD)
- GPU VRAM used vs total
- Per-process CPU and memory usage
- Systemd service status indicators
- NVMe SMART health status
- Connection tracking count vs max

### homelab-logs.json — Homelab Log Overview (13 panels)
Hosts: atlas, unraid, claudebox | Datasource: Loki

Sections:
- Last log received per host (green/yellow/red staleness)
- 24h error count per host
- Error & warning rate over time — all hosts on one graph
- Top 15 noisiest containers (horizontal bar chart)
- Top 8 container error rate over time (time-series)
- Per-host recent errors & warnings (3 side-by-side log panels)
- Container errors live feed (signature-deduped)

### unraid-system.json — Unraid System Dashboard V3 (28 panels)
Host: Unraid | Datasources: InfluxDB + Loki

Sections:
- UPS row: online/battery status, battery %, load %, runtime remaining, daily/monthly power cost
- System stats row: uptime, RAM total/used%, processes, docker container count, net in/out
- CPU usage time-series + system load (1/5/15 min)
- Memory breakdown (used/cached/buffered)
- Per-core CPU heatmap
- Network traffic bidirectional graph
- Disk I/O bytes and IOPS (bidirectional)
- CPU/system temperatures
- Disk space per path (bar gauge)
- Drive temperatures by device — all HDDs (SMART, time-series with last/max table)
- Docker CPU usage per container
- Docker memory usage per container
- Log error/warning rate bar chart (Loki syslog)
- Syslog live panel filtered to errors/warnings
