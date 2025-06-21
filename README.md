# ObservabilityStack

A complete observability stack using Docker Compose with Grafana, Tempo, Loki, Prometheus, and OpenTelemetry Collector. This stack provides unified logging, metrics, and tracing capabilities with TraceQL metrics support.

## 🏗️ Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Applications  │───▶│ OpenTelemetry    │───▶│     Tempo       │
│                 │    │   Collector      │    │   (Traces)      │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                │                        │
                                ▼                        ▼
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│    Promtail     │───▶│      Loki        │    │   Prometheus    │
│  (Log Agent)    │    │     (Logs)       │    │   (Metrics)     │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                │                        ▲
                                ▼                        │
                       ┌──────────────────┐             │
                       │     Grafana      │─────────────┘
                       │ (Visualization)  │
                       └──────────────────┘
```

## 🚀 Components

- **Grafana** (Port 3000): Visualization and dashboards
- **Tempo** (Port 3200): Distributed tracing with TraceQL metrics support
- **Loki** (Port 3100): Log aggregation
- **Prometheus** (Port 9090): Metrics collection and storage
- **OpenTelemetry Collector** (Ports 4317/4318): Telemetry data collection
- **Promtail**: Log collection agent

## ✨ Features

- 📊 **Unified Observability**: Logs, metrics, and traces in one stack
- 🔍 **TraceQL Metrics**: Generate metrics from traces using TraceQL queries
- 🔗 **Correlation**: Link traces to logs and metrics
- 📈 **Real-time Monitoring**: Live dashboards and alerting
- 🐳 **Docker Compose**: Easy deployment and management
- 🔧 **Production Ready**: Proper configuration for production workloads

## 🏃‍♂️ Quick Start

### Prerequisites

- Docker and Docker Compose installed
- At least 4GB RAM available
- Ports 3000, 3100, 3200, 4317, 4318, 8888, 8889, 9090, 13133 available

### 1. Clone and Start

```bash
git clone <your-repo-url>
cd ObservabilityStack
docker-compose up -d
```

### 2. Access Services

- **Grafana**: http://localhost:3000 (admin/admin)
- **Prometheus**: http://localhost:9090
- **Tempo**: http://localhost:3200
- **Loki**: http://localhost:3100

### 3. Verify Setup

```bash
# Check all services are running
docker-compose ps

# Check logs if needed
docker-compose logs [service-name]
```

## 📊 Using TraceQL Metrics

This stack includes TraceQL metrics support, allowing you to generate metrics from traces.

### Example TraceQL Metrics Queries

In Grafana's Explore view with Tempo data source:

```traceql
# Overall request rate
{ } | rate()

# Request rate by service
{ } | rate() by(resource.service.name)

# Error rate
{ status = error } | rate()

# Success rate by service and status
{ status != error } | rate() by(resource.service.name, status)

# Request duration histogram
{ span.kind = "server" } | histogram()

# Rate of specific operations
{ name = "GET /api/users" } | rate()
```

### Grafana Data Source Configuration

The stack includes pre-configured data sources:

- **Tempo**: `http://tempo:3200`
- **Loki**: `http://loki:3100`
- **Prometheus**: `http://prometheus:9090`

## 📁 Configuration Files

| File | Purpose |
|------|---------|
| `docker-compose.yml` | Main orchestration file |
| `tempo-config.yml` | Tempo configuration with TraceQL metrics |
| `loki-config.yml` | Loki log aggregation configuration |
| `prometheus-config.yml` | Prometheus metrics configuration |
| `otel-collector-config.yml` | OpenTelemetry Collector configuration |
| `promtail-config.yml` | Promtail log collection configuration |
| `grafana/provisioning/` | Grafana data source provisioning |

## 🔧 Configuration Details

### Tempo Configuration

- **TraceQL Metrics**: Enabled with `local-blocks` processor
- **Storage**: Local filesystem with proper permissions
- **Retention**: 1 hour for testing (configurable)
- **Metrics Generation**: Sends metrics to Prometheus via remote-write

### OpenTelemetry Collector

- **Receivers**: OTLP (gRPC: 4317, HTTP: 4318), Prometheus
- **Processors**: Batch, memory limiter, resource
- **Exporters**: Tempo (traces), Loki (logs), Prometheus (metrics), Debug

### Prometheus

- **Remote Write**: Enabled for receiving metrics from Tempo
- **Retention**: 200 hours
- **Scrape Targets**: OTel Collector metrics

## 🔍 Sending Data

### Traces (OpenTelemetry)

```bash
# OTLP gRPC endpoint
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317

# OTLP HTTP endpoint  
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
```

### Logs

Promtail automatically collects logs from `/var/log` directory. To send custom logs to Loki:

```bash
curl -X POST http://localhost:3100/loki/api/v1/push \
  -H "Content-Type: application/json" \
  -d '{"streams": [{"stream": {"job": "test"}, "values": [["'$(date +%s%N)'", "test log message"]]}]}'
```

### Metrics

Send metrics to the OpenTelemetry Collector or directly to Prometheus.

## 🛠️ Troubleshooting

### Common Issues

1. **Port Conflicts**
   ```bash
   # Check what's using the ports
   lsof -i :3000,3100,3200,4317,4318,9090
   ```

2. **Permission Issues with Tempo**
   ```bash
   # Fix tempo-data permissions
   chmod 777 tempo-data/
   ```

3. **Container Won't Start**
   ```bash
   # Check logs
   docker-compose logs [service-name]
   
   # Restart specific service
   docker-compose restart [service-name]
   ```

4. **TraceQL Metrics Not Working**
   - Ensure Tempo has `local-blocks` processor enabled
   - Check Prometheus is receiving remote-write data
   - Verify traces are being sent to Tempo

### Health Checks

```bash
# Check Tempo health
curl http://localhost:3200/ready

# Check Prometheus targets
curl http://localhost:9090/api/v1/targets

# Check Loki health
curl http://localhost:3100/ready

# Check OTel Collector metrics
curl http://localhost:8888/metrics
```

## 📈 Monitoring the Stack

### Key Metrics to Monitor

- **Tempo**: `tempo_ingester_live_traces`, `tempo_request_duration_seconds`
- **Loki**: `loki_ingester_streams`, `loki_request_duration_seconds`  
- **Prometheus**: `prometheus_tsdb_head_samples_appended_total`
- **OTel Collector**: `otelcol_receiver_accepted_spans_total`

### Grafana Dashboards

Import these dashboard IDs for monitoring:
- Tempo: 16050
- Loki: 13407
- Prometheus: 3662
- OpenTelemetry Collector: 15983

## 🔒 Security Considerations

- Change default Grafana credentials in production
- Use proper authentication for external access
- Configure network policies for Kubernetes deployments
- Enable TLS for production environments

## 🚀 Production Deployment

For production use:

1. **Use External Storage**: Configure object storage (S3, GCS, Azure)
2. **Scale Components**: Use multiple replicas
3. **Add Monitoring**: Monitor the monitoring stack itself
4. **Configure Retention**: Set appropriate data retention policies
5. **Secure Access**: Add authentication and authorization
6. **Resource Limits**: Set CPU and memory limits

## 📚 Further Reading

- [Grafana Documentation](https://grafana.com/docs/)
- [Tempo Documentation](https://grafana.com/docs/tempo/)
- [Loki Documentation](https://grafana.com/docs/loki/)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [OpenTelemetry Documentation](https://opentelemetry.io/docs/)
- [TraceQL Documentation](https://grafana.com/docs/tempo/latest/traceql/)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test the configuration
5. Submit a pull request

## 📄 License

This project is open source. See LICENSE file for details.

---

**Happy Observing!** 🔍📊📈
