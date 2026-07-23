(how-to-airbyte-observe-airbyte)=

# Observe Airbyte

This guide describes how to wire Airbyte into the [Canonical Observability Stack](https://charmhub.io/topics/canonical-observability-stack) (COS) to get logs, a Grafana dashboard, and a metrics path.

Airbyte exposes three observability endpoints:

- `logging` (`loki_push_api`) - forwards logs from all Airbyte containers to Loki
- `grafana-dashboard` (`grafana_dashboard`) - provisions the prebuilt **Airbyte** dashboard
- `send-otlp` (`otlp`) - points Airbyte's Micrometer exporter at a related OpenTelemetry Collector

Airbyte relates to an [`opentelemetry-collector-k8s`](https://charmhub.io/opentelemetry-collector-k8s) charm deployed alongside it, which in turn relates to COS. The collector is a single, dedicated egress point for all telemetry leaving the model.

```{note}
The Airbyte dashboard is **log-based**: it derives its signals from the forwarded logs. The `send-otlp` metrics path is wired and correct, but carries no data on Airbyte's community edition, which does not emit application metrics over OTLP (native metrics are gated behind Airbyte Enterprise). The metrics path activates automatically on a metrics-enabled edition.
```

## Deploy the collector and relate Airbyte to it

Deploy the collector in the same model as Airbyte:

```bash
juju deploy opentelemetry-collector-k8s --channel 2/stable --trust opentelemetry-collector
```

Relate each of Airbyte's three observability endpoints to the matching collector endpoint:

```bash
juju integrate airbyte-k8s:logging          opentelemetry-collector:receive-loki-logs
juju integrate airbyte-k8s:grafana-dashboard opentelemetry-collector:grafana-dashboards-consumer
juju integrate airbyte-k8s:send-otlp        opentelemetry-collector:receive-otlp
```

Check the wiring with `juju status --relations`:

```text
Integration provider                                 Requirer                                             Interface          Type
airbyte-k8s:grafana-dashboard                        opentelemetry-collector:grafana-dashboards-consumer  grafana_dashboard  regular
opentelemetry-collector:receive-loki-logs            airbyte-k8s:logging                                  loki_push_api      regular
opentelemetry-collector:receive-otlp                 airbyte-k8s:send-otlp                                otlp               regular
```

## Relate the collector to COS

The simplest setup deploys the three COS charms in the same model as Airbyte and the collector:

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

For a production setup, deploy [COS Lite](https://charmhub.io/cos-lite) in a dedicated model and relate the collector to its offers instead of local COS charms; only the three collector-to-COS commands above change, to `juju integrate opentelemetry-collector:<endpoint> <offer-name>`. The collector's endpoint names are the same either way.

## Access the dashboard

Retrieve the Grafana admin password:

```bash
juju run grafana-k8s/0 get-admin-password
```

Log in to Grafana and open the **Airbyte** dashboard. It includes log-level distribution, error and warning rates, log volume by component, Temporal connectivity errors, and raw log panels.

## Verify the wiring

In Loki (or the dashboard's log panels), logs should stream from the Airbyte containers, labeled with Juju topology and a `pebble_service` label per container.

For the full list of endpoints, dashboard panels, and the metrics caveat, see {ref}`Observability <reference-airbyte-observability>`.
