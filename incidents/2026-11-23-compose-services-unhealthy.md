# OpenTelemetry Demo — Post-Mortem Runbook

## Issue

The `product-catalog`, `checkout`, and `payment` services were unhealthy or unreachable. This caused frontend requests to fail with gRPC connection errors.

## Root Causes Identified

### 1. PostgreSQL Initialization Failure

- **Error:** `FATAL: role "root" does not exist`
- **Cause:** The container was running as the `root` user, while PostgreSQL expected the `postgres` user.
- **Fix:** Added `user: postgres` to the `astronomy-db` service in `compose.yaml`.

### 2. Product Catalog Resource Starvation

- **Symptom:** The service was marked unhealthy despite responding to some requests.
- **Original configuration:**
  - Memory limit: `20M`
  - `GOMEMLIMIT`: `16MiB`
  - Healthcheck `start_period`: `10s`, insufficient time for the database connection to initialize
- **Fixes:**
  - Increased memory limit from `20M` to `256M`
  - Increased `GOMEMLIMIT` from `16MiB` to `48MiB`
  - Extended `start_period` from `10s` to `120s`
  - Increased healthcheck retries from `20` to `50`
  - Changed the `astronomy-db` dependency condition from `service_started` to `service_healthy`

### 3. Checkout Service Memory Exhaustion

- **Symptom:** The service became unhealthy under load.
- **Original configuration:** Memory limit of `20M`, matching the original product catalog limit.
- **Fixes:**
  - Increased memory limit from `20M` to `128M`
  - Increased `GOMEMLIMIT` from `48MiB` to `100MiB`
  - Extended `start_period` from `10s` to `60s`
  - Increased healthcheck retries from `20` to `30`
  - Changed the `product-catalog` dependency condition from `service_started` to `service_healthy`

### 4. Payment Service Healthcheck Too Aggressive

- **Symptom:** The service was running and processing transactions but was marked unhealthy.
- **Original configuration:** A `10s` start period and a strict socket timeout.
- **Fixes:**
  - Extended `start_period` from `10s` to `60s`
  - Increased the healthcheck interval from `5s` to `10s`
  - Increased healthcheck retries from `20` to `30`

### 5. Service Dependency Ordering

- **Symptom:** Checkout waited for payment to become healthy while payment was stuck.
- **Original configuration:** `payment` used `condition: service_healthy`.
- **Fix:** Changed the condition to `service_started` to prevent a circular dependency.

## Files Modified

- `compose.yaml` — updated four services with memory, `GOMEMLIMIT`, healthcheck, and dependency changes
- `entrypoint.sh` — created during debugging; unused

## Changes Summary

| Service | Memory | `GOMEMLIMIT` | Start period | Retries | Key change |
|---|---:|---:|---:|---:|---|
| `product-catalog` | `20M` → `256M` | `16MiB` → `48MiB` | `10s` → `120s` | `20` → `50` | More time for database connection |
| `checkout` | `20M` → `128M` | `48MiB` → `100MiB` | `10s` → `60s` | `20` → `30` | Improved load handling |
| `payment` | `140M` (unchanged) | — | `10s` → `60s` | `20` → `30` | More time for healthcheck |
| `astronomy-db` | — | — | — | — | Added `user: postgres` |

## Prevention

- Monitor memory usage and alert when services approach their limits.
- Measure service startup time before setting healthcheck timings.
- Use `service_started` for less critical dependencies where waiting for `service_healthy` is unnecessary.
- Allocate appropriate resources; Go services typically need `128M` or more in production.
- Validate healthchecks independently before deployment.

## Verification

All services report a healthy or running status:

```bash
docker compose ps | grep -E "healthy|running"
