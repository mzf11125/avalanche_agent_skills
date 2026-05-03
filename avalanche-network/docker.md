# Docker Setup for Avalanche

## Single Node (Docker)

```bash
docker run -d \
  --name avalanchego \
  -p 9650:9650 \
  -p 9651:9651 \
  -v avalanchego-data:/root/.avalanchego \
  avaplatform/avalanchego:latest \
  --network-id=fuji \
  --http-host=0.0.0.0 \
  --public-ip-resolution-service=opendns
```

## Docker Compose: Node + Relayer + Monitoring

```yaml
# docker-compose.yml
version: "3.8"

services:
  avalanchego:
    image: avaplatform/avalanchego:latest
    container_name: avalanchego
    ports:
      - "9650:9650"
      - "9651:9651"
    volumes:
      - avalanchego-data:/root/.avalanchego
      - ./configs/avalanchego.json:/root/.avalanchego/configs/node.json
    command: --config-file=/root/.avalanchego/configs/node.json
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9650/ext/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  awm-relayer:
    image: avaplatform/awm-relayer:latest
    container_name: awm-relayer
    volumes:
      - ./configs/relayer-config.json:/config.json
    command: --config-file /config.json
    restart: unless-stopped
    depends_on:
      avalanchego:
        condition: service_healthy
    ports:
      - "9090:9090"  # Prometheus metrics

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9091:9090"
    volumes:
      - ./configs/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
      - ./configs/grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./configs/grafana/datasources:/etc/grafana/provisioning/datasources
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    restart: unless-stopped

volumes:
  avalanchego-data:
  prometheus-data:
  grafana-data:
```

## AvalancheGo Config (configs/avalanchego.json)

```json
{
  "network-id": "fuji",
  "http-host": "0.0.0.0",
  "http-port": 9650,
  "staking-port": 9651,
  "db-dir": "/root/.avalanchego/db",
  "log-level": "info",
  "public-ip-resolution-service": "opendns",
  "http-allowed-hosts": "*",
  "track-subnets": "YOUR_SUBNET_ID"
}
```

## Prometheus Config (configs/prometheus.yml)

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: avalanchego
    static_configs:
      - targets: ['avalanchego:9650']
    metrics_path: /ext/metrics

  - job_name: awm-relayer
    static_configs:
      - targets: ['awm-relayer:9090']
```

## Commands

```bash
# Start all services
docker compose up -d

# View logs
docker compose logs -f avalanchego
docker compose logs -f awm-relayer

# Stop
docker compose down

# Update images
docker compose pull
docker compose up -d
```

## Firewall Rules

```bash
# Allow P2P staking port (required for validator)
ufw allow 9651/tcp

# Allow HTTP API (only if needed externally)
ufw allow 9650/tcp

# Allow Grafana
ufw allow 3000/tcp
```
