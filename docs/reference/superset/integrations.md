(reference-superset-integrations)=

# Integrations

## Relation endpoints

### Required

| Endpoint | Interface | Description |
|---|---|---|
| `postgresql_db` | `postgresql_client` | Metadata database: users, roles, database connections, datasets, dashboards, and the action log. Provided by [postgresql-k8s](https://charmhub.io/postgresql-k8s) or [postgresql](https://charmhub.io/postgresql). |
| `redis` | `redis` | Results cache and Celery message broker. Provided by [redis-k8s](https://charmhub.io/redis-k8s). |

Both endpoints accept a single relation, and the charm stays `blocked` until both exist. Every application of a deployment - UI, worker, and beat - integrates with the same PostgreSQL and Redis applications.

### Optional

| Endpoint | Interface | Description |
|---|---|---|
| `nginx-route` | `nginx_route` | Exposes the web server (port 8088) through [nginx-ingress-integrator](https://charmhub.io/nginx-ingress-integrator), with TLS terminated at the ingress. |
| `trino-catalog` | `trino_catalog` | Database connections created automatically from [trino-k8s](https://charmhub.io/trino-k8s) catalogs, with per-relation credentials. |
| `logging` | `loki_push_api` | Forwards workload logs to Loki. |

### Provided

| Endpoint | Interface | Description |
|---|---|---|
| `metrics-endpoint` | `prometheus_scrape` | Prometheus scrape configuration for the metrics exporter on port 9102. |
| `grafana-dashboard` | `grafana_dashboard` | Ships the "Superset Metrics" dashboard to Grafana. |

## Charm-managed resources in Superset

Through the Trino integration, the charm creates resources inside Superset itself:

| Resource | Naming pattern | Example |
|---|---|---|
| Database connection | `<Catalog title-cased> (<catalog>)` | `Google Ads (google_ads)` |
| Permission grant | `database_access` on the connection, granted to the role named by `self-registration-role` | `database_access` granted to `Public` |

The charm only creates connections and updates their URI when the Trino host, port, username, or credentials change. It never renames or deletes a connection, and never touches connections created by administrators. See {ref}`Integrate with Trino <how-to-superset-integrate-with-trino>`.

## Juju secrets used by the charm

| Secret | Owner | Purpose |
|---|---|---|
| SMTP credentials (user-supplied, any name) | User | SMTP host, credentials, and external URL for alerts and reports; referenced by the `smtp-secret-id` configuration option and granted to each application. |
| Trino relation credentials | Trino charm | Per-relation Trino username and password, granted to Superset over the `trino-catalog` relation. Rotation is picked up automatically. |

The Superset secret key and initial admin password are plain configuration options (`superset-secret-key` and `admin-password`), not Juju secrets. Treat the secret key as sensitive and keep it identical across the UI, worker, and beat applications of one deployment.
