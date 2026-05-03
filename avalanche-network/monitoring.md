# Prometheus & Grafana Monitoring

## AvalancheGo Metrics

AvalancheGo exposes Prometheus metrics at:
```
http://localhost:9650/ext/metrics
```

### Key Metrics

| Metric | Description |
|--------|-------------|
| `avalanche_network_peers` | Number of connected peers |
| `avalanche_C_blks_accepted_count` | C-Chain blocks accepted |
| `avalanche_C_blks_processing` | Blocks currently being processed |
| `avalanche_C_vm_eth_rpc_duration_sum` | RPC call duration |
| `avalanche_P_vm_total_staked` | Total staked AVAX |
| `avalanche_health_checks_failing_count` | Number of failing health checks |
| `go_memstats_alloc_bytes` | Memory usage |
| `process_cpu_seconds_total` | CPU usage |

### Prometheus Scrape Config

```yaml
scrape_configs:
  - job_name: avalanchego
    static_configs:
      - targets: ['localhost:9650']
    metrics_path: /ext/metrics
    scrape_interval: 15s
```

## Grafana Dashboard

### Datasource (configs/grafana/datasources/prometheus.yml)

```yaml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    url: http://prometheus:9090
    isDefault: true
```

### Key Panels to Build

**Node Health Panel:**
```promql
avalanche_health_checks_failing_count
```

**Peer Count:**
```promql
avalanche_network_peers
```

**Block Rate (C-Chain):**
```promql
rate(avalanche_C_blks_accepted_count[5m])
```

**Memory Usage:**
```promql
go_memstats_alloc_bytes / 1024 / 1024
```

**CPU Usage:**
```promql
rate(process_cpu_seconds_total[1m]) * 100
```

## Alerting Rules

```yaml
# prometheus-alerts.yml
groups:
  - name: avalanchego
    rules:
      - alert: NodeUnhealthy
        expr: avalanche_health_checks_failing_count > 0
        for: 2m
        annotations:
          summary: "AvalancheGo node has failing health checks"

      - alert: NoPeers
        expr: avalanche_network_peers < 3
        for: 5m
        annotations:
          summary: "Node has fewer than 3 peers"

      - alert: HighMemoryUsage
        expr: go_memstats_alloc_bytes > 28e9
        for: 5m
        annotations:
          summary: "Node memory usage above 28GB"

      - alert: BlocksStuck
        expr: rate(avalanche_C_blks_accepted_count[10m]) == 0
        for: 10m
        annotations:
          summary: "C-Chain not accepting blocks"
```

Add to prometheus.yml:
```yaml
rule_files:
  - /etc/prometheus/prometheus-alerts.yml
```

## Pre-built Dashboards

The Avalanche team provides Grafana dashboards:
- [AvalancheGo Grafana Dashboard](https://github.com/ava-labs/avalanchego/tree/master/scripts/grafana)

Import via Grafana UI: Dashboards → Import → Upload JSON file.

## Log Monitoring

```bash
# Follow logs with systemd
journalctl -u avalanchego -f

# Filter for errors
journalctl -u avalanchego | grep -i "error\|fatal\|panic"

# Check bootstrap progress
journalctl -u avalanchego | grep -i "bootstrap"
```
