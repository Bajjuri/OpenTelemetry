# OpenTelemetry Dice Server

A small Express + TypeScript app (`/rolldice`) instrumented with the OpenTelemetry Node SDK.
Traces and metrics are exported over OTLP/HTTP to a local Grafana stack (Tempo for traces,
Prometheus for metrics) running in a container.

## How it fits together

```
app (OpenTelemetry SDK) --OTLP/HTTP :4318--> grafana/otel-lgtm container
                                               |- Tempo       (traces)
                                               |- Prometheus  (metrics)
                                               |- Grafana UI  http://localhost:3000
```

## Status

Verified working end to end: traces and metrics from `/rolldice` show up in Grafana.
Tested on Windows 11 with Podman running on a WSL2 machine (`podman-machine-default`).
Podman needs `--pids-limit=0` on this setup (see Troubleshooting).

## Prerequisites

- Node.js and npm
- [Podman](https://podman.io/) (Docker works too: replace `podman` with `docker` and drop `--pids-limit=0`)
  - On Windows, Podman needs WSL2: `wsl --install`, then reboot
  - `podman machine init` and `podman machine start`

## Setup

```powershell
npm install
```

## Run

### 1. Start Grafana (terminal 1)

```powershell
podman run --rm --pids-limit=0 --name lgtm -p 3000:3000 -p 4317:4317 -p 4318:4318 docker.io/grafana/otel-lgtm:latest
```

Wait until the log reports it is ready (the first start takes a minute or two).

- `3000`: Grafana UI
- `4317` / `4318`: OTLP gRPC / HTTP receivers (the app uses 4318)
- `--pids-limit=0` works around a `pids` cgroup error with Podman on WSL2; it is not needed on Docker
- The container stores data in memory only, so traces and metrics are lost when it stops

### 2. Start the app (terminal 2)

```powershell
npm start
```

The app listens on http://localhost:8080 (override with the `PORT` environment variable).

### 3. Generate traffic (terminal 3)

```powershell
curl.exe localhost:8080/rolldice
```

Returns a number from 1 to 6. Run it a few times.

### 4. View telemetry

Open http://localhost:3000 and log in with `admin` / `admin` (you can skip the password change).

- **Traces:** Explore, choose the **Tempo** data source, use the Search query type, and filter by
  service name `dice-server`.
- **Metrics:** Explore, choose the **Prometheus** data source.

Traces appear after a few seconds. Metrics are exported every 10 seconds.

### Stop

1. Stop the app with `Ctrl+C` in its terminal.
2. Stop the Grafana container: press `Ctrl+C` in its terminal, or run `podman stop lgtm`.
   The container was started with `--rm`, so it is removed and its traces and metrics are lost.
3. Optional, to free memory (roughly 1-2 GB): stop the Podman VM.

   ```powershell
   podman machine stop
   ```

   Images are kept, so the next start is quick. If WSL still holds memory afterwards, run
   `wsl --shutdown`.

To start again: `podman machine start`, then the Grafana command from step 1 of Run.

## Scripts

| Script | What it does |
| --- | --- |
| `npm start` | Runs the app with OpenTelemetry instrumentation loaded |
| `npm run dev` | Runs the app without instrumentation |
| `npm run typecheck` | Type-checks with `tsc` |

## Configuration

The exporter endpoint is set in [src/instrumentation.ts](src/instrumentation.ts). It defaults to
`http://localhost:4318` and can be changed with an environment variable:

```powershell
$env:OTEL_EXPORTER_OTLP_ENDPOINT = "http://collector.example:4318"
npm start
```

To print spans to the terminal instead, swap in `ConsoleSpanExporter` (from
`@opentelemetry/sdk-trace-node`) and `ConsoleMetricExporter` (from `@opentelemetry/sdk-metrics`).

## Troubleshooting

- **No traces in Grafana:** confirm the container is running (`podman ps`) and that the app prints
  no export errors. Check that the app is hit at least once.
- **`crun: controller pids is not available`:** add `--pids-limit=0` to `podman run`.
- **`npm run curl` fails:** `curl` is not an npm script; run `curl.exe` directly.
- **protobufjs install script skipped:** harmless for this setup. To allow it, run
  `npm install-scripts approve protobufjs`.
