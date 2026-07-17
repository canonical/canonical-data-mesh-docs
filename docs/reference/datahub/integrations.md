(reference-datahub-integrations)=

# Integrations

## Relation endpoints

### Required

| Endpoint | Interface | Description |
|---|---|---|
| `db` | `postgresql_client` | Metadata storage. Provided by [postgresql](https://charmhub.io/postgresql) or [postgresql-k8s](https://charmhub.io/postgresql-k8s). |
| `kafka` | `kafka_client` | Event streaming for metadata change events and audit logging. Provided by [kafka](https://charmhub.io/kafka) or [kafka-k8s](https://charmhub.io/kafka-k8s). |
| `opensearch` | `opensearch_client` | Search and graph indexing. Provided by [opensearch](https://charmhub.io/opensearch) (machine charm). |

All three are commonly consumed as cross-model offers, since OpenSearch runs on a machine cloud. Each endpoint supports a single relation.

### Optional

| Endpoint | Interface | Description |
|---|---|---|
| `frontend-ingress` | `ingress` | Exposes the web UI (port 9002). Required for SSO. |
| `gms-ingress` | `ingress` | Exposes the GMS API (port 8080) for external clients. |
| `trino-catalog` | `trino_catalog` | Automatic metadata ingestion from [trino-k8s](https://charmhub.io/trino-k8s) catalogs. |
| `oauth` | `oauth` | OIDC credentials for SSO, from the Canonical Identity Platform or an [oauth-external-idp-integrator](https://charmhub.io/oauth-external-idp-integrator). |
| `logging` | `loki_push_api` | Forwards logs from all three containers to Loki. |

### Provided

| Endpoint | Interface | Description |
|---|---|---|
| `metrics-endpoint` | `prometheus_scrape` | Prometheus scrape configuration for the GMS and frontend JMX exporters. |
| `grafana-dashboard` | `grafana_dashboard` | Ships the "DataHub Monitoring" dashboard to Grafana. |

## Charm-managed resources in DataHub

Through the Trino integration, the charm creates resources inside DataHub. They are identifiable by naming convention, and user-created resources that do not match these patterns are never touched:

| Resource | Naming pattern | Example |
|---|---|---|
| Ingestion sources | `[juju] <catalog>-ingestion` | `[juju] sales-ingestion` |
| Per-catalog password secrets | `JUJU_MANAGED_TRINO_PASSWORD_<NORMALIZED_CATALOG>` | catalog `my-catalog.test` becomes `JUJU_MANAGED_TRINO_PASSWORD_MY_CATALOG_TEST` |
| GMS access token secret | `JUJU_MANAGED_GMS_TOKEN` | - |

Catalog names are normalized by uppercasing and replacing non-alphanumeric characters with `_`. When a catalog is removed or the Trino relation is broken, the corresponding ingestion sources and secrets are deleted automatically; ingested metadata is preserved.

## Juju secrets used by the charm

| Secret | Owner | Purpose |
|---|---|---|
| `datahub-encryption-keys` (user-supplied, any name) | User | GMS and frontend encryption keys; referenced by the `encryption-keys-secret-id` configuration option and granted to the charm. |
| `datahub-init-pwd` | Charm | The auto-generated initial admin password, returned by the `get-password` action. |
| `datahub-system-client-secret` | Charm | Internal system client credentials for GMS. |
| `datahub-ingestion-token` | Charm | Access token used by charm-managed ingestion sources. |
