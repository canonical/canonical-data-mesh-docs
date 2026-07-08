# Observe DataHub

This guide describes how to wire DataHub into the [Canonical Observability Stack](https://charmhub.io/topics/canonical-observability-stack) (COS) to get metrics, a Grafana dashboard, alert rules, and logs.

DataHub exposes three observability endpoints:

- `metrics-endpoint` - Prometheus scrape targets for the GMS and frontend JMX exporters
- `grafana-dashboard` - a prebuilt "DataHub Monitoring" dashboard
- `logging` - log forwarding from all three containers to Loki

## Relate to the observability charms

The simplest setup deploys the three COS charms in the same model as DataHub:

```bash
juju deploy prometheus-k8s --channel 2/stable --trust
juju deploy loki-k8s --channel 2/stable --trust
juju deploy grafana-k8s --channel 2/stable --trust

juju integrate datahub-k8s:metrics-endpoint prometheus-k8s
juju integrate datahub-k8s:logging loki-k8s
juju integrate datahub-k8s:grafana-dashboard grafana-k8s
```

Register Prometheus and Loki as Grafana data sources so the dashboard can render:

```bash
juju integrate prometheus-k8s:grafana-source grafana-k8s:grafana-source
juju integrate loki-k8s:grafana-source grafana-k8s:grafana-source
```

For a production setup, deploy [COS Lite](https://charmhub.io/cos-lite) in a dedicated model and consume its offers instead; the DataHub side of the relations stays the same.

## Access the dashboard

Retrieve the Grafana admin password and address:

```bash
juju run grafana-k8s/0 get-admin-password
```

Log in to Grafana and open the **DataHub Monitoring** dashboard. It includes availability, GMS operations (Kafka consumer lag, database ingest latency, and OpenSearch latencies), JVM health, and per-container log panels.

## Verify the wiring

In Prometheus, the query `up{juju_application="datahub-k8s"}` should return `1` for both the GMS and frontend scrape jobs. The two targets are distinguished by job name, not port: the `instance` label carries Juju topology.

In Loki (or the dashboard's log panels), logs should stream from all three containers: `datahub-gms`, `datahub-frontend`, and `datahub-actions`.

## Alerts

The charm ships four Prometheus alert rules covering service availability, JVM heap starvation, excessive garbage collection, and Kafka consumer lag. They load automatically over the `metrics-endpoint` relation. See [Observability](../../reference/datahub/observability.md) for the full list, thresholds, and metric names.
