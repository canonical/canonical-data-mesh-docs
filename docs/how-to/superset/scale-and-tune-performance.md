(how-to-superset-scale-and-tune-performance)=

# Scale and tune performance

This guide describes how to add asynchronous query execution and scheduled tasks to a Superset deployment, scale each function independently, and tune process-level concurrency.

## Deploy workers for asynchronous queries

Workers run long queries, scheduled reports, and cache warm-ups in the background, keeping the UI responsive. A worker is the same charm deployed with `charm-function=worker`:

```bash
juju deploy superset-k8s --channel 6/stable --config charm-function=worker \
  --config superset-secret-key=<SECRET_KEY> \
  --config admin-password=<ADMIN_PASSWORD> \
  superset-k8s-worker

juju integrate superset-k8s-worker postgresql-k8s
juju integrate superset-k8s-worker redis-k8s
```

```{important}
Workers must share the same `superset-secret-key` and `admin-password` as the
UI application, and integrate with the same PostgreSQL and Redis charms.
Otherwise they cannot decrypt stored connection credentials.
```

Asynchronous query execution is then enabled per database in the UI: edit the database connection and check **Asynchronous query execution** under **Performance**. Recommended for all production databases.

For asynchronous results to reach the browser, enable the `GLOBAL_ASYNC_QUERIES` feature flag on the UI and worker applications:

```bash
juju config superset-k8s feature-flags=GLOBAL_ASYNC_QUERIES
juju config superset-k8s-worker feature-flags=GLOBAL_ASYNC_QUERIES
```

See {ref}`Supported feature flags <reference-superset-feature-flags>` for the full list.

## Deploy the beat scheduler

The beat scheduler triggers periodic tasks such as cache warm-ups, SQL Lab query cleanup, and the daily pruning of the action log. Deploy exactly one beat application, and scale it to a single unit: multiple schedulers produce duplicate task runs.

```bash
juju deploy superset-k8s --channel 6/stable --config charm-function=beat \
  --config superset-secret-key=<SECRET_KEY> \
  --config admin-password=<ADMIN_PASSWORD> \
  superset-k8s-beat

juju integrate superset-k8s-beat postgresql-k8s
juju integrate superset-k8s-beat redis-k8s
```

## Scale the applications

The UI and worker applications scale horizontally:

```bash
juju scale-application superset-k8s -n 3
juju scale-application superset-k8s-worker -n 5
```

Three units each of the UI and worker applications is a reasonable starting point for production. Scale workers independently when the deployment is dominated by asynchronous queries or scheduled reports.

## Tune process concurrency

Beyond pod count, each pod's internal concurrency is configurable:

| Option | Applies to | Effect |
|---|---|---|
| `server-worker-amount` | UI | Gunicorn worker processes per pod (1-32). |
| `gunicorn-timeout` | UI | Gunicorn request timeout in seconds (30-600). |
| `webserver-timeout` | UI | How long the web server keeps a database connection open, in seconds (60-300). |
| `celery-worker-concurrency` | Worker | Celery processes per worker pod (0-128, `0` uses the Celery default). |
| `sqlalchemy-pool-size` | All | Connections kept in the metadata database pool. |
| `sqlalchemy-max-overflow` | All | Extra connections allowed beyond the pool size. |

An example profile for three UI pods and five worker pods:

```bash
juju config superset-k8s server-worker-amount=2 gunicorn-timeout=120
juju config superset-k8s-worker celery-worker-concurrency=4
```

```{note}
Start conservative and increase gradually. Each Gunicorn and Celery process
holds its own database connections, so raising concurrency raises the load on
PostgreSQL and Redis as well. Watch queue depth and database connection counts
while you tune - see {ref}`Observe Superset <how-to-superset-observe-superset>`.
```

## Warm up the dashboard cache

Cache warm-up pre-renders charts so the first user of the day does not pay the query cost. It runs daily at 07:01 UTC over the ten most-viewed dashboards of the past week, as a beat-scheduled task executed by a worker, so it needs both a beat and a worker application:

```bash
juju config superset-k8s cache-warmup=true
juju config superset-k8s redis-timeout=600
```

`redis-timeout` sets how long cached results stay valid, in seconds.

## Prune the action log

Superset records every user action in the `logs` table of its metadata database, which grows without bound on a busy deployment. The charm schedules a daily pruning task, also executed through beat and worker:

```bash
juju config superset-k8s log-retention-days=180
```

Set `log-retention-enabled=false` to stop pruning entirely. The default retention is 730 days.
