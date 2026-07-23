(reference-airbyte-observability)=

# Observability

Reference for the telemetry Airbyte exposes over its observability relations. For setup instructions, see {ref}`Observe Airbyte <how-to-airbyte-observe-airbyte>`.

## Observability relations

The charm integrates with the [Canonical Observability Stack](https://charmhub.io/topics/canonical-observability-stack) (COS) over three relations, typically through an [`opentelemetry-collector-k8s`](https://charmhub.io/opentelemetry-collector-k8s) charm:

| Endpoint | Interface | Direction | Content |
|---|---|---|---|
| `logging` | `loki_push_api` | requires | Forwards logs from all Airbyte containers to Loki |
| `grafana-dashboard` | `grafana_dashboard` | provides | Provisions the **Airbyte** dashboard |
| `send-otlp` | `otlp` | requires | Sends Airbyte's Micrometer metrics to an OTLP endpoint |

## Grafana dashboard

The `grafana-dashboard` relation ships the **Airbyte** dashboard. It is log-based: every panel derives its signal from the forwarded container logs rather than from metrics.

| Panel | Content |
|---|---|
| Log level distribution | Breakdown of log lines by level (`INFO`, `WARNING`, `ERROR`, and so on) |
| Error & warning rate | Rate of error and warning log lines over time |
| Log volume by component | Log throughput split per Airbyte component |
| Temporal connectivity errors | Log lines indicating loss of connectivity to Temporal |
| Errors and exceptions | Filtered panel of error and exception log lines |
| All logs | Raw, unfiltered log stream |

The dashboard uses the standard COS datasource and Juju topology variables, so it filters per model, application, and unit.

## Metrics

The `send-otlp` relation points Airbyte's Micrometer exporter at the OTLP endpoint of a related OpenTelemetry Collector.

```{note}
The metrics path is wired and correct, but carries **no data on Airbyte's community edition**: community does not emit application metrics over OTLP (native metrics are gated behind Airbyte Enterprise). The metrics activate automatically on a metrics-enabled edition. Airbyte's per-sync detail (records, bytes, job status) is written to job logs in object storage and exposed through the Airbyte API, not the OTLP stream.
```

## Logs

The `logging` relation forwards Pebble-captured logs from all of the charm's containers to Loki, labeled with Juju topology and a `pebble_service` label per container:

`airbyte-bootloader`, `airbyte-connector-builder-server`, `airbyte-cron`, `airbyte-pod-sweeper`, `airbyte-server`, `airbyte-workers`, `airbyte-workload-api-server`, and `airbyte-workload-launcher`.
