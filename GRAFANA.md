# Grafana LGTM Stack (docker-otel-lgtm)

The OpenTelemetry collector and the Grafana LGTM stack (Loki, Grafana, Tempo, Prometheus, plus Pyroscope) are up and running. Readiness marker: `/tmp/ready` inside the container.

## Access

| Service | URL / Port | Notes |
|---|---|---|
| Grafana | http://localhost:3000 | User: `admin`, Password: `admin` |
| OTLP gRPC endpoint | `localhost:4317` | OpenTelemetry collector |
| OTLP HTTP endpoint | `localhost:4318` | OpenTelemetry collector |
| Tempo | `localhost:3200` | Traces |
| Pyroscope | `localhost:4040` | Profiles |
| Prometheus | `localhost:9090` | Metrics |

## Startup Time Summary

| Component | Startup |
|---|---|
| Grafana | 3 seconds |
| Loki | 2 seconds |
| Prometheus | 1 second |
| Tempo | 1 second |
| Pyroscope | 3 seconds |
| OpenTelemetry collector | 1 second |
| **Total** | **3 seconds** |

## AI Tool Integration (MCP)

- **Tempo MCP:** server disabled; enable with `TEMPO_EXTRA_ARGS=--query-frontend.mcp-server.enabled=true`
- **Grafana MCP:** server enabled with service account token
- **Claude Code:** `bash <(docker exec lgtm cat /etc/lgtm/claude-mcp-setup.sh)`
- **Other tools:** `docker exec lgtm cat /etc/lgtm/mcp.json`
- **Docs:** https://github.com/grafana/docker-otel-lgtm/blob/v0.35.0/docs/mcp-integration.md
