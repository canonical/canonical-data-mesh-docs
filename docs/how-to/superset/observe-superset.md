(how-to-superset-observe-superset)=

# Observe Superset

This guide describes how to connect Superset to the [Canonical Observability Stack](https://charmhub.io/topics/canonical-observability-stack) (COS) to collect its metrics, logs, and a prebuilt dashboard.

## Deploy COS

COS usually runs in its own model. Deploy the `cos-lite` bundle and expose the endpoints Superset consumes as offers:

```bash
juju add-model cos
juju deploy cos-lite --trust

juju offer prometheus:metrics-endpoint
juju offer loki:logging
juju offer grafana:grafana-dashboard
```

## Integrate Superset with COS

Switch back to the Superset model and integrate each endpoint:

```bash
juju switch <SUPERSET_MODEL>

juju integrate superset-k8s admin/cos.prometheus
juju integrate superset-k8s admin/cos.loki
juju integrate superset-k8s admin/cos.grafana
```

Repeat for the worker and beat applications to collect their telemetry too. Worker applications export Celery task metrics, which the dashboard uses for its worker panels.

## Verify

Retrieve the Grafana admin password and the Grafana address:

```bash
juju run grafana/0 -m cos get-admin-password
juju status -m cos
```

Open Grafana on port 3000 of its unit address and log in as `admin`. The **Superset Metrics** dashboard is provisioned automatically and shows availability, dashboard and chart load times, SQL Lab query timings, worker status, and log panels for the UI, worker, and beat applications.

For what each metric and alert means, see {ref}`Observability <reference-superset-observability>`.
