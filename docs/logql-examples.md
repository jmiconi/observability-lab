# LogQL troubleshooting examples

These examples assume labels such as `environment` and `source` are attached during ingestion.

## All Windows Event Log entries

```logql
{source="windows_eventlog"}
```

## Errors containing a keyword

```logql
{source="windows_eventlog"} |= "error"
```

## Search several failure terms

```logql
{source="windows_eventlog"} |~ "(?i)(error|failed|critical|timeout)"
```

## Restrict to an environment

```logql
{environment="lab", source="windows_eventlog"}
```

## Count matching failures over time

```logql
sum(count_over_time({source="windows_eventlog"} |~ "(?i)(error|failed)" [5m]))
```

## Incident workflow

1. Start with the narrow incident time range.
2. Filter by environment/source.
3. Search for the symptom or service name.
4. Expand the time range to identify the first occurrence.
5. Correlate with configuration changes or adjacent services.
