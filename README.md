# Pi 5 Monitoring Stack

Lightweight monitoring for Raspberry Pi 5 using Prometheus, Grafana, and Node Exporter.

## Stack

| Service | Port | Purpose |
|---------|------|---------|
| Grafana | 9300 | Dashboards and visualization |
| Prometheus | 9090 | Metrics storage and querying |
| Node Exporter | 9100 | System metrics (CPU, RAM, disk, network) |
| Smartctl Exporter | 9902 | HDD/SSD SMART health data |

## Metrics Collected

### System (Node Exporter)
- CPU usage and load
- Memory and swap usage
- Disk space and I/O stats
- Network traffic
- CPU temperature
- Pressure Stall Information (PSI) — requires `psi=1` kernel parameter

### Storage Health (Smartctl Exporter)
- SMART health status (`smartprom_smart_passed`)
- Drive temperature
- Power-on hours
- Reallocated sectors (bad sectors)
- Pending sectors

## Quick Start

```bash
docker compose up -d
```

## Access

- Grafana: `http://raspberrypi.local:9300`
- Prometheus: `http://raspberrypi.local:9090`

Default Grafana credentials are in `.env`.

## Grafana Setup

1. Add Prometheus data source: `http://prometheus:9090`
2. Import dashboard **1860** (Node Exporter Full)
3. Create custom panels for SMART metrics

## Useful PromQL Queries

```promql
# CPU temperature
node_thermal_zone_temp

# Memory used %
100 * (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)

# Disk used %
100 - ((node_filesystem_avail_bytes{mountpoint="/"} * 100) / node_filesystem_size_bytes{mountpoint="/"})

# HDD health (1 = healthy)
smartprom_smart_passed

# HDD temperature
smartprom_temperature_celsius_raw

# HDD lifespan (years)
smartprom_power_on_hours_raw / 8760
```

## Enabling PSI (Pressure Metrics)

Edit `/boot/firmware/cmdline.txt`, add `psi=1` to the end of the line, then reboot.

## Data Retention

Prometheus retains metrics for 30 days. Adjust in `docker-compose.yml`:

```yaml
--storage.tsdb.retention.time=30d
```

## Resource Usage

| Service | RAM |
|---------|-----|
| Grafana | ~250MB |
| Prometheus | ~70MB |
| Node Exporter | ~15MB |
| Smartctl Exporter | ~15MB |
| **Total** | ~350MB |

## Files

```
.
├── docker-compose.yml    # Service definitions
├── prometheus.yml        # Scrape targets
├── .env                  # Grafana credentials (gitignored)
└── .gitignore
```

## Notes

- Containers auto-restart on reboot (`restart: unless-stopped`)
- Node Exporter uses `network_mode: host` for accurate host metrics
- Prometheus reaches host services via `host.docker.internal`
- Smartctl Exporter requires `privileged: true` to access `/dev`
