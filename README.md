# ci-container-monitoring-lab

A small, fully local DevOps stack that wires a single commit all the way to a
running, monitored service:

**commit → CI → container → deploy → dashboard**

The stack contains:

- **app/** — a Node.js + Express service exposing `/`, `/health`, `/metrics` and `/work`
- **Dockerfile** — builds the service image
- **docker-compose.yml** — runs the app, Prometheus and Grafana together
- **prometheus/** — Prometheus scrape configuration
- **grafana/** — provisioned datasource and a "Service Overview" dashboard
- **.github/workflows/ci.yml** — GitHub Actions pipeline (install, test, build, smoke test)

Everything runs locally and for free. No cloud account or billing is required.

## Prerequisites

- Docker + Docker Compose
- Node.js 18+ (only needed if you want to run tests outside a container)
- `git`, `curl`

## Run the stack

```bash
docker compose up -d          # build + start app, Prometheus, Grafana
docker compose ps             # show container status and health
docker compose logs -f app    # follow the application logs
```

Services once up:

| Service     | URL                      |
|-------------|--------------------------|
| App         | http://localhost:8080    |
| Prometheus  | http://localhost:9090    |
| Grafana     | http://localhost:3000    |

Grafana logs in anonymously (admin/admin also works). Open the **Service
Overview** dashboard under the **Lab** folder.

Tear down:

```bash
docker compose down
```

## Verify the service

```bash
curl localhost:8080/health     # -> {"status":"ok"}
curl localhost:8080/           # -> service metadata
curl localhost:8080/metrics    # -> Prometheus metrics
```

Check the Prometheus scrape target:

```
http://localhost:9090/targets   # the app target should be UP
```

## Generate sample traffic

```bash
./scripts/generate-traffic.sh                       # defaults to localhost:8080
./scripts/generate-traffic.sh http://localhost:8080 500
```

Then watch the panels in the Grafana **Service Overview** dashboard update.

## Run the tests locally

```bash
cd app
npm install
npm test
```

## GitHub Actions

The workflow in `.github/workflows/ci.yml` runs automatically on every push and
pull request. It installs dependencies, runs the unit tests, builds the Docker
image and smoke-tests the running container. Watch it under the **Actions** tab
of your fork, or trigger it by pushing a commit / opening a PR.

## Rollback

Releases are tagged in Git. You can inspect history with:

```bash
git log --oneline
git tag
```

To roll a bad release back to a previous version, either revert the offending
commit and redeploy, or rebuild from a previous tag:

```bash
git revert <commit>          # revert a bad release
docker compose up -d --build # redeploy the reverted version
```

## Cloud mapping

The same local flow maps cleanly to a managed cloud release without changing the
application behavior:

1. Build the image in CI and push it to Artifact Registry.
2. Deploy that image to Cloud Run with a service configuration that exposes the
   same `PORT` and health endpoint, then route traffic through the public URL.
3. Use the Cloud Monitoring metrics endpoint and dashboards to observe request
   count, latency, and health signals from the running service.

In practice, the local equivalents are:

- Local image build: `docker build -t ci-container-monitoring-lab:local .`
- Cloud build: `gcloud builds submit --tag REGION-docker.pkg.dev/PROJECT_ID/REPO/ci-container-monitoring-lab`
- Local deploy: `docker compose up -d`
- Cloud deploy: `gcloud run deploy ci-container-monitoring-lab --image REGION-docker.pkg.dev/PROJECT_ID/REPO/ci-container-monitoring-lab --region REGION`
- Local observability: Prometheus + Grafana dashboards
- Cloud observability: Cloud Monitoring + Cloud Run metrics / logs

This is a written mapping only; it does not require billing or a live cloud deployment.
