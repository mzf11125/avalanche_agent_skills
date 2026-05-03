# Relayer Monitoring

## Prometheus Metrics

The relayer exposes Prometheus metrics at `http://localhost:9090/metrics` (configurable via `metrics-port`).

### Key Metrics

| Metric | Description |
|--------|-------------|
| `awm_relayer_messages_received_total` | Total messages received from source chains |
| `awm_relayer_messages_delivered_total` | Total messages successfully delivered |
| `awm_relayer_messages_failed_total` | Total delivery failures |
| `awm_relayer_delivery_latency_seconds` | Histogram of message delivery latency |
| `awm_relayer_signature_aggregation_latency_seconds` | Time to collect validator signatures |
| `awm_relayer_destination_balance` | Relayer account balance on each destination chain |

### Prometheus Scrape Config

```yaml
# prometheus.yml
scrape_configs:
  - job_name: awm-relayer
    static_configs:
      - targets: ['localhost:9090']
```

## Health Check

The relayer exposes a health endpoint:
```bash
curl http://localhost:8080/health
# {"status":"ok"}
```

## Critical Alerts

Set up alerts for:

```yaml
# alerting rules
- alert: RelayerDown
  expr: up{job="awm-relayer"} == 0
  for: 2m

- alert: RelayerDeliveryFailures
  expr: rate(awm_relayer_messages_failed_total[5m]) > 0.1
  for: 5m

- alert: RelayerLowBalance
  expr: awm_relayer_destination_balance < 0.1
  for: 1m
  annotations:
    summary: "Relayer account balance low on {{ $labels.chain }}"
```

## Common Failure Modes

**Messages not being picked up:**
- Check relayer is running: `docker ps` or `systemctl status awm-relayer`
- Check relayer logs for errors
- Verify source chain WebSocket connection is active
- Verify the Teleporter contract address in config matches the deployed address

**Delivery failures:**
- Check relayer account has sufficient balance on destination chain
- Check `requiredGasLimit` is high enough (increase if destination tx reverts)
- Check destination chain RPC is reachable

**Signature aggregation failures:**
- Validators may be offline - check validator uptime
- P-Chain API may be unreachable - check `p-chain-api` config
- Insufficient stake weight signing - need 67%+ of stake weight

## Log Analysis

```bash
# Follow relayer logs
docker logs -f awm-relayer

# Filter for errors
docker logs awm-relayer 2>&1 | grep -i error

# Filter for a specific message ID
docker logs awm-relayer 2>&1 | grep "0xYourMessageID"
```
