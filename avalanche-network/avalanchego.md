# AvalancheGo Setup

## System Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| CPU | 8 cores | 16 cores |
| RAM | 16 GB | 32 GB |
| Storage | 1 TB SSD | 2 TB NVMe SSD |
| Network | 5 Mbps | 25 Mbps |
| OS | Ubuntu 20.04+ | Ubuntu 22.04 |

## Installation

### Option 1: Install Script (Recommended)

```bash
wget -nd -m https://raw.githubusercontent.com/ava-labs/avalanche-docs/master/scripts/avalanchego-installer.sh
chmod 755 avalanchego-installer.sh
./avalanchego-installer.sh
```

### Option 2: Binary

```bash
# Get latest version
VERSION=$(curl -s https://api.github.com/repos/ava-labs/avalanchego/releases/latest | grep tag_name | cut -d'"' -f4)
wget https://github.com/ava-labs/avalanchego/releases/download/${VERSION}/avalanchego-linux-amd64-${VERSION}.tar.gz
tar xvf avalanchego-linux-amd64-${VERSION}.tar.gz
sudo mv avalanchego-${VERSION}/avalanchego /usr/local/bin/
```

### Option 3: Build from Source

```bash
git clone https://github.com/ava-labs/avalanchego
cd avalanchego
./scripts/build.sh
```

## Running AvalancheGo

### Mainnet

```bash
avalanchego \
  --network-id=mainnet \
  --http-host=0.0.0.0 \
  --http-port=9650 \
  --staking-port=9651 \
  --db-dir=/data/avalanchego \
  --log-level=info
```

### Fuji Testnet

```bash
avalanchego \
  --network-id=fuji \
  --http-host=0.0.0.0 \
  --http-port=9650 \
  --staking-port=9651 \
  --db-dir=/data/avalanchego-fuji \
  --log-level=info
```

## Key Configuration Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--network-id` | `mainnet` | Network: `mainnet`, `fuji`, or custom |
| `--http-host` | `127.0.0.1` | HTTP API bind address |
| `--http-port` | `9650` | HTTP API port |
| `--staking-port` | `9651` | P2P staking port (must be open to internet) |
| `--db-dir` | `~/.avalanchego/db` | Database directory |
| `--log-level` | `info` | Log level: `debug`, `info`, `warn`, `error` |
| `--bootstrap-ips` | auto | Bootstrap node IPs |
| `--public-ip` | auto | Your node's public IP |
| `--http-allowed-hosts` | `localhost` | Allowed HTTP hosts (set `*` for remote access) |

## Config File

Instead of flags, use a JSON config file:

```json
{
  "network-id": "fuji",
  "http-host": "0.0.0.0",
  "http-port": 9650,
  "staking-port": 9651,
  "db-dir": "/data/avalanchego",
  "log-level": "info",
  "public-ip-resolution-service": "opendns",
  "http-allowed-hosts": "*"
}
```

```bash
avalanchego --config-file=/etc/avalanchego/config.json
```

## Systemd Service

```ini
# /etc/systemd/system/avalanchego.service
[Unit]
Description=AvalancheGo Node
After=network.target

[Service]
User=avalanche
ExecStart=/usr/local/bin/avalanchego --config-file=/etc/avalanchego/config.json
Restart=always
RestartSec=3
LimitNOFILE=32768

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable avalanchego
sudo systemctl start avalanchego
sudo journalctl -u avalanchego -f
```

## Check Node Health

```bash
curl -X POST http://localhost:9650/ext/health \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"health.health","params":{},"id":1}'
```

A healthy node returns `"healthy": true`.

## Check Bootstrap Status

```bash
curl -X POST http://localhost:9650/ext/info \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"info.isBootstrapped","params":{"chain":"C"},"id":1}'
```
