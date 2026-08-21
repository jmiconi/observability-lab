# Infrastructure Observability Lab

A reproducible lab for **centralized Windows infrastructure logging** using Grafana Alloy, Loki and Grafana.

The project is intentionally evidence-driven: the documented flow was rebuilt from a clean Ubuntu Server VM and a clean Windows Server VM, then validated with a controlled Windows Event Log entry retrieved through Loki and Grafana.

> **Validation status: PASS — end-to-end tested.** See [`VALIDATION.md`](VALIDATION.md).

## What this project demonstrates

- Centralized Windows Event Log collection
- Grafana Alloy as a Windows service
- Loki single-node log storage
- Grafana datasource provisioning
- LogQL troubleshooting queries
- Persistent Windows Event Log bookmarks
- Low-cardinality label design
- Docker Compose deployment from a clean Linux baseline
- Reproducibility testing and documented failure findings

## Validated architecture

```text
Windows Server
  System + Application Event Logs
            |
            v
      Grafana Alloy
            |
            | HTTP push
            v
          Loki
            |
            v
         Grafana
            |
            v
          LogQL
```

Validated chain:

```text
Windows Event Log
  -> Grafana Alloy 1.18.1
  -> Loki 3.7.0
  -> Grafana 11.5.2
  -> LogQL result
```

## Repository structure

```text
observability-lab/
├── alloy/
│   └── windows-events.alloy
├── grafana/
│   └── provisioning/
│       └── datasources/
│           └── loki.yml
├── loki/
│   └── loki.yml
├── docs/
│   ├── architecture.md
│   └── logql-examples.md
├── compose.yml
├── VALIDATION.md
└── README.md
```

## Tested baseline

Server side:

- Ubuntu Server 24.04.4 LTS
- Docker 29.1.3
- Docker Compose 2.40.3
- Loki 3.7.0
- Grafana 11.5.2

Windows source:

- Windows Server 2022 Standard Evaluation
- Grafana Alloy 1.18.1

## Quick start — backend

Install Docker on Ubuntu 24.04:

```bash
sudo apt update
sudo apt install -y docker.io docker-compose-v2
sudo systemctl enable --now docker
```

Clone and start the stack:

```bash
git clone https://github.com/jmiconi/observability-lab.git
cd observability-lab
docker compose config
docker compose up -d
```

Validate Loki readiness from the host:

```bash
for i in $(seq 1 30); do
  code=$(curl -s -o /tmp/loki-ready.txt -w '%{http_code}' http://localhost:3100/ready || true)
  printf 'Attempt %02d: HTTP %s - ' "$i" "$code"
  cat /tmp/loki-ready.txt 2>/dev/null || true
  echo
  [ "$code" = "200" ] && break
  sleep 2
done
```

Validate Grafana:

```bash
curl -s http://localhost:3000/api/health
```

Grafana provisions Loki automatically as its default datasource.

## Windows event collection

Install Grafana Alloy on a Windows Server or Windows client, then adapt [`alloy/windows-events.alloy`](alloy/windows-events.alloy) so the Loki endpoint points to the lab server.

The validated base configuration collects the broadly available channels:

- `System`
- `Application`

The configuration also uses persistent bookmarks so restarts continue from the previous read position.

After replacing the Alloy configuration, restart the service:

```powershell
Restart-Service Alloy
Get-Service Alloy
```

## Controlled end-to-end test

Create a recognizable Windows Application event:

```powershell
eventcreate `
  /T INFORMATION `
  /ID 100 `
  /L APPLICATION `
  /SO ObservabilityLab `
  /D "OBS-LAB-E2E Windows Event Log to Alloy to Loki"
```

Query it directly from Loki:

```bash
curl -sG http://localhost:3100/loki/api/v1/query_range \
  --data-urlencode 'query={job="windows-eventlog",channel="Application"} |= "OBS-LAB-E2E"' \
  --data-urlencode 'since=1h' \
  --data-urlencode 'limit=20'
```

Equivalent Grafana Explore query:

```logql
{job="windows-eventlog", channel="Application"} |= "OBS-LAB-E2E"
```

The clean validation run returned the controlled event in both Loki and Grafana.

## Important findings from validation

- The Loki 3.7.0 image used in the lab does not provide `/bin/sh`; a `CMD-SHELL` healthcheck therefore fails even when Loki itself starts correctly.
- Loki `/ready` returns HTTP 503 briefly during normal ingester startup before becoming HTTP 200.
- The Alloy Windows installer may still be finalizing immediately after silent invocation; validate the installed service and executable after installation settles.
- Event Log channel label values are case-sensitive in LogQL. The observed values were `Application` and `System`, not lower-case equivalents.

## Label model

The Windows configuration intentionally uses stable, low-cardinality labels such as:

```text
host
environment
job
source
channel
service
```

Do not turn event IDs, messages, usernames or other high-cardinality fields into labels unless there is a strong operational reason.

## LogQL examples

See [`docs/logql-examples.md`](docs/logql-examples.md) for tested query patterns.

## Engineering principles

- Logs are operational evidence, not just storage.
- Collection configuration should be version-controlled.
- Labels should make navigation easier without creating excessive cardinality.
- Raw evidence should remain accessible behind dashboards and alerts.
- A documented deployment is not considered complete until it is reproduced from a clean baseline.

## Security

- Do not publish production hostnames, IP addresses, credentials or customer data.
- Restrict Loki and Grafana exposure according to the deployment environment.
- Review Windows Event Log payloads for sensitive content before centralizing them.
- Change default lab credentials before using this pattern outside an isolated test environment.

## Validation

Full clean-lab evidence, versions, findings and scope are documented in [`VALIDATION.md`](VALIDATION.md).

Current status:

```text
PASS: Windows Event Log -> Alloy -> Loki -> Grafana -> LogQL
```

## Roadmap

- Linux and Docker log collection with Alloy
- Metrics integration with Prometheus
- IIS parsing examples
- Network/exporter telemetry
- Alert rules
- Incident-oriented Grafana dashboards
