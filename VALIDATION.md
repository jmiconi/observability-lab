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

## Loki startup and healthcheck finding

A first local Loki 3.7.0 container test showed that Loki itself starts correctly, but a shell-based Docker healthcheck is invalid for the tested image.

Observed behavior:

- Loki reached `Loki started` in the container logs.
- The host could reach `http://localhost:3100/ready`.
- During startup `/ready` temporarily returned HTTP 503 with `Ingester not ready: waiting for 15s after being ready`, which is normal readiness behavior during initialization.
- Docker nevertheless marked the container unhealthy because the configured `CMD-SHELL` healthcheck attempted to execute `/bin/sh` inside the Loki image.
- The tested `grafana/loki:3.7.0` image does not provide `/bin/sh`, producing `stat /bin/sh: no such file or directory` on every healthcheck attempt.

Therefore the lab must not use a shell-based in-container healthcheck for Loki 3.7.0. Readiness will instead be verified through Loki's `/ready` endpoint from the host or another container with an HTTP client, while Compose startup ordering must not depend on the invalid shell healthcheck.

## Validation goal

The lab will only be marked PASS when the following chain is demonstrated end-to-end:

```text
Windows Event Log
  -> Grafana Alloy
  -> Loki
  -> Grafana / LogQL
```

A controlled Windows event must be generated and retrieved from Loki through the documented workflow before the repository claims reproducibility.

## Current status

```text
PASS: clean Ubuntu baseline and Docker runtime
PASS: Loki 3.7.0 process startup and host-side /ready reachability
CONFIRMED GAP: original repository has no runnable Loki/Grafana stack
CONFIRMED FINDING: shell-based Loki healthcheck is incompatible with tested image
IN PROGRESS: reproducible Loki + Grafana backend and Windows Alloy ingestion
```
