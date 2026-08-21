# LogQL troubleshooting examples

These examples match the labels observed during the clean Windows Event Log validation run.

## All Windows Event Log entries

```logql
{job="windows-eventlog"}
```

## Application log only

The Windows Event Log source exposes the channel label using the channel's canonical name, so label matching is case-sensitive:

```logql
{job="windows-eventlog", channel="Application"}
```

## System log only

```logql
{job="windows-eventlog", channel="System"}
```

## Search for a controlled validation event

```logql
{job="windows-eventlog", channel="Application"} |= "OBS-LAB-E2E"
```

## Errors containing a keyword

```logql
{job="windows-eventlog"} |= "error"
```

## Search several failure terms

```logql
{job="windows-eventlog"} |~ "(?i)(error|failed|critical|timeout)"
```

## Restrict to an environment

```logql
{environment="lab", job="windows-eventlog"}
```

## Restrict to a host

```logql
{environment="lab", job="windows-eventlog", host="LAB-WIN01"}
```

Replace `LAB-WIN01` with the host label present in your environment.

## Count matching failures over time

```logql
sum(count_over_time({job="windows-eventlog"} |~ "(?i)(error|failed)" [5m]))
```

## Validation finding: label case

During validation, an initial query using `channel="application"` returned no result even though ingestion was working. Loki exposed the source channel as `Application`, so this query succeeded:

```logql
{job="windows-eventlog", channel="Application"} |= "OBS-LAB-E2E"
```

Treat label values as case-sensitive when troubleshooting an apparently empty query.

## Incident workflow

1. Start with the narrow incident time range.
2. Verify available labels and series before assuming ingestion is broken.
3. Filter by environment, host, job and channel.
4. Search for the symptom, event source or service name.
5. Expand the time range to identify the first occurrence.
6. Correlate with configuration changes or adjacent services.
