# Observability

Reference for the telemetry Charmed DataHub exposes over its observability relations. For setup instructions, see [Observe DataHub](../../how-to/datahub/observe-datahub.md).

## Metrics endpoints

The GMS and frontend rocks embed the JMX Prometheus exporter, enabled by the charm. Both services share the pod network namespace, so they use distinct ports:

| Target | Port | Content |
|---|---|---|
| `datahub-gms` | 4318 | JVM metrics and DataHub GMS application metrics |
| `datahub-frontend` | 4319 | JVM metrics |
| `datahub-actions` | - | No metrics endpoint (logs only) |

Prometheus distinguishes the two targets by scrape job name; the `instance` label carries Juju topology, not the host and port.

## Metric namespaces

| Prefix | Source | Examples |
|---|---|---|
| `java_lang_*` | JVM MBeans | `java_lang_Memory_HeapMemoryUsage_used`, `java_lang_Threading_ThreadCount` |
| `jvm_*` | JMX exporter built-in collectors | `jvm_gc_collection_seconds_sum` |
| `metrics_com_linkedin_metadata_*` | DataHub application (Dropwizard) | Kafka consumer lag, aspect ingest latency, OpenSearch bulk-processor rates |

## Grafana dashboard

The `grafana-dashboard` relation ships the **DataHub Monitoring** dashboard with four rows:

| Row | Panels |
|---|---|
| DataHub availability | Availability stat per metrics target |
| GMS Operations | Kafka consumer lag, database ingest latency, Elasticsearch search latency, Elasticsearch bulk writes |
| JVM | Heap usage ratio, heap used, GC time rate, thread count, process CPU load |
| Logs | Per-container log panels: GMS, frontend, and actions |

The dashboard uses the standard COS datasource and Juju topology variables, so it filters per model, application, and unit.

## Alert rules

Four Prometheus alert rules load automatically over the `metrics-endpoint` relation:

| Alert | Severity | Condition |
|---|---|---|
| `DatahubServiceDown` | critical | A DataHub metrics target has been unreachable (`up == 0`) for more than 5 minutes. |
| `DatahubHeapStarvation` | warning | A JVM has been above 90% heap usage for 10 minutes. |
| `DatahubExcessiveGC` | warning | A JVM spent more than 50% of wall-clock time in garbage collection over the last 5 minutes, sustained for 10 minutes. |
| `DatahubKafkaConsumerLag` | warning | End-to-end lag of the embedded metadata consumers (MCL/MCP) has been above 5 minutes for 15 minutes; search and graph indices are falling behind ingestion. |

## Logs

The `logging` relation forwards Pebble-captured logs from all three containers (`datahub-gms`, `datahub-frontend`, `datahub-actions`) to Loki, labeled with Juju topology and the `pebble_service` label per container.
