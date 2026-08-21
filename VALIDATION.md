# Validation

## Clean-lab baseline

Validation is being performed from a clean **Ubuntu Server 24.04.4 LTS** virtual-machine baseline.

Observed baseline:

- Git 2.43.0 present
- Docker not present initially
- `/opt` empty before cloning the repository

Docker was installed from the Ubuntu 24.04 repositories:

```bash
sudo apt update
sudo apt install -y docker.io docker-compose-v2
sudo systemctl enable --now docker
```

Observed versions during the run:

- Docker 29.1.3
- Docker Compose 2.40.3

`hello-world` completed successfully and the non-root lab user was added to the `docker` group.

## Repository baseline finding

The repository was cloned at commit:

```text
18e5b5c Add MIT license
```

At that point the repository contained:

```text
README.md
LICENSE
alloy/windows-events.alloy
docs/architecture.md
docs/logql-examples.md
```

No Docker Compose file, Loki configuration, Grafana provisioning, or executable server-side quick-start was present.

This means the original repository described the intended architecture but was **not reproducible from a clean Linux host as documented**.

That gap is being treated as a validation finding rather than silently assumed away.

## Loki and Grafana backend validation

A minimal backend was created with pinned images:

- `grafana/loki:3.7.0`
- `grafana/grafana:11.5.2`

Grafana provisions Loki automatically as its default datasource.

### Healthcheck finding

The first Compose attempt used a Loki `CMD-SHELL` healthcheck. The Loki 3.7.0 image used in this run does not provide `/bin/sh`, so Docker marked the container unhealthy even though Loki itself had started successfully.

Observed Docker healthcheck error:

```text
exec: "/bin/sh": stat /bin/sh: no such file or directory
```

The container logs independently showed:

```text
Loki started
```

Loki readiness was therefore moved out of the container healthcheck and validated explicitly through the host HTTP endpoint.

Immediately after startup `/ready` returned HTTP 503 with:

```text
Ingester not ready: waiting for 15s after being ready
```

This is expected during Loki startup. In the clean run, readiness became HTTP 200 on the ninth two-second polling attempt.

Final backend state:

```text
obs-lab-loki      Up
obs-lab-grafana   Up (healthy)
```

Loki:

```text
GET /ready -> HTTP 200
ready
```

Grafana:

```json
{
  "database": "ok",
  "version": "11.5.2"
}
```

This validates the clean deployment of the server-side observability backend.

## Validation goal

The lab will only be marked PASS when the following chain is demonstrated end-to-end:

```text
Windows Event Log
  -> Grafana Alloy
  -> Loki
  -> Grafana / LogQL
```

A controlled Windows event must be generated and retrieved from Loki through the documented workflow before the repository claims full reproducibility.

## Current status

```text
PASS: clean Ubuntu baseline and Docker runtime
PASS: Loki 3.7.0 + Grafana 11.5.2 backend from Compose
PASS: Loki datasource provisioned automatically in Grafana
CONFIRMED FINDING: Loki image cannot use the attempted CMD-SHELL healthcheck
IN PROGRESS: Windows Alloy ingestion and LogQL retrieval
```
