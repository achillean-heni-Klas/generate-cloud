# monitoring-agent

Lightweight Linux system monitoring agent for modern infrastructure. Collects metrics and forwards to monitoring backends.

## Features

- **Minimal**: Single binary, low resource usage
- **Performance**: Go-based for efficient collection
- **Configurable**: Flexible metric filtering
- **Cloud Native**: Works in containers and VMs

## Getting Started

### Requirements
- Go 1.19+
- Linux (Ubuntu, CentOS, Alpine tested)

### Build & Run

```bash
# Setup environment
export GOPATH=$HOME/go
export GOROOT=/usr/local/go

# Get source
mkdir -p $GOPATH/src/github.com/sys-monitor
cd $GOPATH/src/github.com/sys-monitor
git clone https://github.com/sys-monitor/agent.git
cd agent

# Build
go mod download
go build -o monitoring-agent cmd/agent/main.go

# Run
./monitoring-agent --config config.json
```

### Docker Deployment
```bash
docker run -d \
  --name monitoring-agent \
  -v /proc:/host/proc:ro \
  -v /sys:/host/sys:ro \
  -p 1988:1988 \
  sysmonitor/agent:latest
```

## Configuration

Example `config.json`:

```json
{
  "heartbeat": {
    "enabled": true,
    "addr": "monitor.service:8433",
    "interval": 60
  },
  "collector": {
    "enabled": true,
    "endpoints": [
      "collector1.service:6060",
      "collector2.service:6060"
    ]
  },
  "filters": {
    "skip_metrics": [
      "cpu.steal",
      "cpu.guest"
    ],
    "skip_disks": [
      "tmpfs",
      "devtmpfs"
    ]
  },
  "api": {
    "enabled": true,
    "port": ":1988"
  }
}
```

## Metrics Collected

### System Resources
- **CPU**: Usage, load, per-core stats
- **Memory**: RAM, swap, cache
- **Disk**: IO, space, inodes
- **Network**: Throughput, packets, errors
- **Processes**: Count, resource usage

### Plugin Support
```bash
# Custom collectors in /etc/monitoring-agent/plugins/
chmod +x /etc/monitoring-agent/plugins/custom.sh
```

### Web Interface
Built-in dashboard at `http://localhost:1988`

![Dashboard](docs/dashboard.png)

## Production Deployment

### Systemd Service
```bash
sudo cp contrib/monitoring-agent.service /etc/systemd/system/
sudo systemctl enable monitoring-agent
sudo systemctl start monitoring-agent
```

### Filter Configuration
```json
{
  "filters": {
    "skip_metrics": [
      "disk.io.read_ops",
      "disk.io.write_ops"
    ],
    "disk_patterns": [
      "loop*",
      "ram*"
    ]
  }
}
```

### Health Endpoints
```bash
# Agent status
curl http://localhost:1988/health

# Current metrics
curl http://localhost:1988/metrics
```

## Architecture

```
[System] → [Agent] → [Collector] → [Backend]
    ↓          ↓          ↓
[procfs]  [Plugins] [Heartbeat]
```

## Resource Usage

- **Memory**: ~45MB RSS
- **CPU**: <0.8% average
- **Network**: Minimal overhead
- **Disk**: No local storage

## Comparison

| Feature | monitoring-agent | Zabbix | Telegraf |
|---------|-----------------|--------|----------|
| Binary | 14MB | 4MB | 75MB |
| Deps | None | Several | Several |
| Config | JSON | Custom | TOML |
| Plugins | ✅ | ✅ | ✅ |
| UI | ✅ | ❌ | ❌ |

## Troubleshooting

**Permission errors accessing /proc**
```bash
sudo ./monitoring-agent --config config.json
```

**Port conflict on 1988**
```bash
# Change in config
{
  "api": {
    "port": ":2000"
  }
}
```

### Debug Logging
```bash
./monitoring-agent --config config.json --verbose
```

## Development

### Build Commands
```bash
make build
make test
make dist
```

### Contributing
1. Fork repository
2. Create feature branch
3. Add tests
4. Submit PR

## Deployment Tools

For fleet management, see [Fleet Manager](http://ops-tools.io/fleet-manager) for automation.

*Practical monitoring for production systems*
