(how-to-datahub-observe-datahub)=

# Observe DataHub

This guide describes how to wire DataHub into the [Canonical Observability Stack](https://charmhub.io/topics/canonical-observability-stack) (COS) to get metrics, a Grafana dashboard, alert rules, and logs.

DataHub exposes three observability endpoints:

- `metrics-endpoint` - Prometheus scrape targets for the GMS and frontend JMX exporters
- `grafana-dashboard` - a prebuilt "DataHub Monitoring" dashboard
- `logging` - log forwarding from all three containers to Loki

DataHub relates to an [`opentelemetry-collector-k8s`](https://charmhub.io/opentelemetry-collector-k8s) charm deployed alongside it, which in turn relates to COS. The collector is a single, dedicated egress point for all telemetry leaving the model.

## Deploy the collector and relate DataHub to it

Deploy the collector in the same model as DataHub:

```bash
juju deploy opentelemetry-collector-k8s --channel 2/stable --trust opentelemetry-collector
```

Relate each of DataHub's three observability endpoints to the matching collector endpoint:

```bash
juju integrate datahub-k8s:metrics-endpoint  opentelemetry-collector:metrics-endpoint
juju integrate datahub-k8s:grafana-dashboard opentelemetry-collector:grafana-dashboards-consumer
juju integrate datahub-k8s:logging           opentelemetry-collector:receive-loki-logs
```

Check the wiring with `juju status --relations`:

```text
Integration provider                                 Requirer                                             Interface           Type
datahub-k8s:grafana-dashboard                        opentelemetry-collector:grafana-dashboards-consumer  grafana_dashboard   regular
datahub-k8s:metrics-endpoint                         opentelemetry-collector:metrics-endpoint             prometheus_scrape   regular
opentelemetry-collector:receive-loki-logs            datahub-k8s:logging                                  loki_push_api       regular
```

## Relate the collector to COS

The simplest setup deploys the three COS charms in the same model as DataHub and the collector:

```bash
juju deploy prometheus-k8s --channel 2/stable --trust
juju deploy loki-k8s       --channel 2/stable --trust
juju deploy grafana-k8s    --channel 2/stable --trust

juju integrate opentelemetry-collector:send-remote-write           prometheus-k8s:receive-remote-write
juju integrate opentelemetry-collector:send-loki-logs              loki-k8s:logging
juju integrate opentelemetry-collector:grafana-dashboards-provider grafana-k8s:grafana-dashboard
```

Register Prometheus and Loki as Grafana data sources so the dashboard can render:

```bash
juju integrate prometheus-k8s:grafana-source grafana-k8s:grafana-source
juju integrate loki-k8s:grafana-source       grafana-k8s:grafana-source
```

For a production setup, deploy [COS Lite](https://charmhub.io/cos-lite) (or a Mimir-based COS
stack) in a dedicated model and relate the collector to its offers instead of local COS charms;
only the three commands above change, to `juju integrate opentelemetry-collector:<endpoint>
<offer-name>`. The collector's endpoint names are the same either way. Mimir deployments use
`receive-remote-write` on the `mimir` offer in place of `prometheus-k8s`; the collector-side
endpoint (`send-remote-write`) doesn't change.

Once everything settles, `juju status --relations` shows the full chain from DataHub through the
collector to COS:

```text
Integration provider                                 Requirer                                             Interface                Type     Message
datahub-k8s:grafana-dashboard                        opentelemetry-collector:grafana-dashboards-consumer  grafana_dashboard        regular
datahub-k8s:metrics-endpoint                         opentelemetry-collector:metrics-endpoint             prometheus_scrape        regular
loki-k8s:grafana-source                              grafana-k8s:grafana-source                           grafana_datasource       regular
loki-k8s:logging                                     opentelemetry-collector:send-loki-logs               loki_push_api            regular
opentelemetry-collector:grafana-dashboards-provider  grafana-k8s:grafana-dashboard                        grafana_dashboard        regular
opentelemetry-collector:receive-loki-logs            datahub-k8s:logging                                  loki_push_api            regular
prometheus-k8s:grafana-source                        grafana-k8s:grafana-source                           grafana_datasource       regular
prometheus-k8s:receive-remote-write                  opentelemetry-collector:send-remote-write            prometheus_remote_write  regular
```

## Access the dashboard

Retrieve the Grafana admin password and address:

```bash
juju run grafana-k8s/0 get-admin-password
```

Log in to Grafana and open the **DataHub Monitoring** dashboard. It includes availability, GMS operations (Kafka consumer lag, database ingest latency, and OpenSearch latencies), JVM health, and per-container log panels.

## Verify the wiring

In Prometheus, the query `up{juju_application="datahub-k8s"}` should return `1` for both the GMS and frontend scrape jobs. The two targets are distinguished by job name, not port: the `instance` label carries Juju topology. The collector forwards these scrapes unchanged, so the query is identical whether DataHub relates to Prometheus directly or through the collector.

In Loki (or the dashboard's log panels), logs should stream from all three containers: `datahub-gms`, `datahub-frontend`, and `datahub-actions`.

## Alerts

The charm ships four Prometheus alert rules covering service availability, JVM heap starvation, excessive garbage collection, and Kafka consumer lag. They load automatically over the `metrics-endpoint` relation and are forwarded by the collector like any other scrape target. See {ref}`Observability <reference-datahub-observability>` for the full list, thresholds, and metric names.
