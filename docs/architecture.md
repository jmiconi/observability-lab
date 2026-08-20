# Architecture

## Pipeline

```text
Windows / Linux hosts
        |
        | logs and metrics
        v
     Grafana Alloy
        |
        +-------------------+
        |                   |
        v                   v
      Loki            Metrics backend
        |                   |
        +---------+---------+
                  |
                  v
               Grafana
```

## Design goals

- Keep collection close to the source.
- Centralize storage and query.
- Use low-cardinality labels for navigation.
- Preserve raw evidence for incident investigation.
- Version-control collection and parsing configuration.

## Windows logging

For Windows systems, Alloy can read native Event Log channels and forward them into Loki. Separate sources can be created for Application, System, Security or product-specific channels depending on the operational need.

## Label strategy

Good labels describe stable dimensions such as environment, source type and service. Values such as usernames, request IDs or full paths are better kept in the log body or structured metadata instead of indexed labels.
