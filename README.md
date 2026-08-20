# Infrastructure Observability Lab

Patterns for building **centralized infrastructure observability** across Windows and Linux systems using Grafana, Loki and Grafana Alloy.

The project focuses on collecting operational evidence that helps answer two questions quickly:

1. **What changed?**
2. **Where is the failure occurring?**

## What this project demonstrates

- Centralized log collection
- Windows Event Log ingestion
- IIS/application log ingestion patterns
- Loki label design
- LogQL troubleshooting queries
- Agent → backend → visualization architecture
- Sanitized infrastructure telemetry examples

## Architecture

```text
 Windows Servers        Linux Services
       |                      |
       | logs                 | logs
       v                      v
 +------------+         +------------+
 |   Alloy    |         |   Alloy    |
 +------+-----+         +------+-----+
        \                    /
         \                  /
          v                v
             +--------+
             |  Loki  |
             +---+----+
                 |
                 v
             +--------+
             | Grafana|
             +--------+
```

## Repository structure

```text
observability-lab/
├── alloy/
│   └── windows-events.alloy
├── docs/
│   ├── architecture.md
│   └── logql-examples.md
└── README.md
```

## Windows event collection

The Alloy example reads selected Windows Event Log channels and forwards them to Loki. Labels are intentionally low-cardinality: environment, host and source.

## LogQL examples

The repository includes queries for:

- filtering errors by host
- reviewing authentication events
- finding application failures
- correlating events around an incident window

## Engineering principles

- Logs are operational evidence, not just storage.
- Labels should help navigation without creating excessive cardinality.
- Collection configuration should be version-controlled.
- Dashboards should link back to the raw evidence.
- Alerting should be added only after useful signals are understood.

## Security

The examples contain no production names, IP addresses, credentials or customer data. Log pipelines must still be reviewed for sensitive payloads before deployment.

## Roadmap

- Metrics integration
- IIS parsing examples
- Network/exporter telemetry
- Alert rules
- Incident-oriented Grafana dashboards
