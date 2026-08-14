(reference-superset-observability)=

# Observability

Reference for the telemetry Superset exposes over its observability relations. For setup instructions, see {ref}`Observe Superset <how-to-superset-observe-superset>`.

## Metrics endpoint

Every unit exposes Prometheus metrics on port 9102. What produces them depends on the application's `charm-function`:

| Application | Exporter | Content |
|---|---|---|
| UI (`app-gunicorn`, `app`) | StatsD exporter | Superset application metrics, emitted by the workload over UDP on port 9125 and translated to Prometheus format |
| Beat (`beat`) | StatsD exporter | Scheduler activity |
| Worker (`worker`) | Celery exporter | Celery worker and task metrics, read from the Redis broker |

## Metric namespaces

| Prefix | Source | Examples |
|---|---|---|
| `superset_*` | Superset StatsD logger | `superset_welcome`, `superset_log`, `superset_DashboardRestApi_get_success`, `superset_ChartDataRestApi_data_time_count`, `superset_sqllab_query_time_executing_query_sum` |
| `celery_*` | Celery exporter on worker units | `celery_worker_up`, task counters and timings |

Superset's REST API metrics follow the pattern `superset_<ApiClass>_<endpoint>_<outcome>` for counters and `superset_<ApiClass>_<endpoint>_time_<sum\|count>` for timings, so per-endpoint latency is the ratio of the two.

## Grafana dashboard

The `grafana-dashboard` relation ships the **Superset Metrics** dashboard with these panels:

| Group | Panels |
|---|---|
| Metrics | Is Superset up?, rate of all Superset events, dashboard loads, chart data loads |
| SQL Lab | SQL Lab query execution time, time to fetch SQL Lab results |
| Timings | Time to load charts, time to get dashboard dataset |
| Workers | Worker count and availability, derived from `celery_worker_up` |
| Logs | Separate panels for UI, worker, and beat logs |

The dashboard filters on Juju topology, so it works per model, application, and unit.

## Alert rules

Three Prometheus alert rules load automatically over the `metrics-endpoint` relation:

| Alert | Severity | Condition |
|---|---|---|
| `SupersetDown` | critical | No Superset instance has served the welcome endpoint for 2 minutes. |
| `SupersetHalfOrMoreDown` | high | Half or more of the Superset instances have stopped serving for 2 minutes. |
| `WorkersDown` | critical | No Celery worker has reported as up for 2 minutes. |

```{note}
The availability rules are traffic-derived: they measure requests served rather
than a health endpoint, so a deployment with no user traffic at all can trigger
`SupersetDown`. Take that into account when tuning thresholds for a deployment
that is idle for long periods.
```

## Logs

The `logging` relation forwards the workload logs of every related application to Loki, labeled with Juju topology. The workload also writes a rotated application log to `/var/log/superset.log` inside the container.
