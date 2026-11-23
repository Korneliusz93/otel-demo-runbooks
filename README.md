# OTel Demo Runbooks

A collection of practical runbooks created while exploring the [OpenTelemetry Demo](https://github.com/open-telemetry/opentelemetry-demo).

The runbooks document incidents, diagnostic evidence, fixes, and verification steps from a local Docker-based learning environment.

## Start the OpenTelemetry Demo with Docker Compose

### Prerequisites

- Docker Desktop with Docker Compose v2
- About 6 GB of RAM and 14 GB of free disk space for the full demo
- The OpenTelemetry Demo repository cloned locally

The resource requirements are from the [official Docker deployment guide](https://opentelemetry.io/docs/demo/docker-deployment/).

### Clone the demo

```bash
git clone https://github.com/open-telemetry/opentelemetry-demo.git
cd opentelemetry-demo
```

### Start the full demo, observability stack, extras, and agent

Run this from the `opentelemetry-demo` directory. This uses Docker Compose directly and does not require `make`.

```bash
docker compose --env-file .env --env-file .env.override \
  -f compose.yaml -f compose.full.yaml \
  -f compose.observability.yaml -f compose.extras.yaml \
  -f compose.agent.yaml \
  up --remove-orphans --detach
```

Keep `.env.override` local if it contains personal settings or credentials. Do not commit it to a public repository.

### Check service status

```bash
docker compose --env-file .env --env-file .env.override \
  -f compose.yaml -f compose.full.yaml \
  -f compose.observability.yaml -f compose.extras.yaml \
  -f compose.agent.yaml \
  ps
```

### Open the demo

- Store: <http://localhost:8080>
- Jaeger: <http://localhost:8080/jaeger/ui/>
- Grafana: <http://localhost:8080/grafana/>
- Feature flags: <http://localhost:8080/feature/>
- Load generator: <http://localhost:8080/loadgen/>
- Demo chatbot: <http://localhost:8080/chatbot/>

### Stop the demo

Run from the same directory with the same Compose files:

```bash
docker compose --env-file .env --env-file .env.override \
  -f compose.yaml -f compose.full.yaml \
  -f compose.observability.yaml -f compose.extras.yaml \
  -f compose.agent.yaml \
  down
```

This removes the demo containers and network. It preserves volumes because the command does not use `-v`.

Runbooks

Runbooks are maintained as separate documents. Add links here as they become available:

<!-- Example: - [Runbook title](URL) — short description. -->

## Official References

- [OpenTelemetry Demo repository](https://github.com/open-telemetry/opentelemetry-demo)
- [Docker deployment guide](https://opentelemetry.io/docs/demo/docker-deployment/)
- [OpenTelemetry Demo documentation](https://opentelemetry.io/docs/demo/
