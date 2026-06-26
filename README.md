# k8s-test-core-server

Go 1.23 / chi — MSA domain API service.

## Status

Bootstrap stage. HTTP server exposing health/readiness endpoints, built and shipped
through Kaniko → GHCR → ArgoCD onto Kubernetes, fronted by the Istio Gateway at
`api.${domain}/v1/core`. Domain logic (entities, persistence, auth, events,
observability) is planned — see Roadmap.

## Ports

| Port | Purpose |
|------|---------|
| `8080` | HTTP API |

## Environment Variables

```bash
HTTP_PORT=8080   # listen port (default 8080)
```

## Local Development

```bash
go mod download
go run ./cmd/server
go test ./...
```

## Build

```bash
docker build --build-arg GIT_SHA=$(git rev-parse --short HEAD) -t core .
```

## API

| Method | Path | Description |
|--------|------|-------------|
| GET | `/healthz` | Liveness probe → `{"status":"ok"}` |
| GET | `/readyz` | Readiness probe → `{"status":"ready"}` |

## Roadmap

Not yet implemented: domain entity CRUD, MySQL persistence (HeatWave), Redis cache,
Kafka event publishing, JWT auth middleware, OpenTelemetry metrics/traces on `:9090`.
