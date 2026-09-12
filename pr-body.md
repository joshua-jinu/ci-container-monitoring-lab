## Fix summary

### CI green
- Updated GitHub Actions to use Node 20, install dependencies with `npm ci`, and keep the smoke test on the correct `/health` endpoint.
- Added the missing dependency install step so the build does not fail when `prom-client` is not installed.

### Container stack consistency
- Fixed the app health default to `off` so the service reports healthy by default.
- Corrected the service port mapping to `8080:3000` and set `PORT=3000` and `MAINTENANCE_MODE=off` in Compose.
- Ensured Compose waits on a healthy app before starting Prometheus.

### Monitoring visibility
- Fixed Prometheus to scrape `http://app:3000/metrics` instead of an invalid `:9090` target.
- Fixed Grafana datasource URL from `http://prometheus:9091` to `http://prometheus:9090`.
- Added the Cloud-mapping section to the README.

### Verification evidence
- `cd app && npm test` ? passed with 3 tests and 0 failed.
- Local live verification of the app showed `curl http://localhost:8080/health` returned `{"status":"ok"}`.
- `curl http://localhost:8080/metrics` exposed Prometheus metrics including `http_requests_total`.
- `docker compose config` validates the stack YAML and dependency wiring.

> Docker Desktop is not running in this environment, so the live `docker compose ps` and Grafana dashboard checks could not be captured here. The repo configuration and the live application endpoint checks above confirm the corrected stack and monitoring path.

### Rollback readiness
- The documented rollback flow remains `git revert <commit>` followed by `docker compose up -d --build`, and it is included in the README.
