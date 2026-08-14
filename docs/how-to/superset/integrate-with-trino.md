(how-to-superset-integrate-with-trino)=

# Integrate with Trino

This guide describes how to connect Superset to [Trino](https://charmhub.io/trino-k8s), the federated SQL query engine of the Canonical Data Mesh. The integration turns every Trino catalog into a Superset database connection automatically, so analysts can chart data from any connected source without an administrator creating connections by hand.

## Prerequisites

- Superset is deployed with `charm-function=app-gunicorn` (the default). Worker and beat applications do not take part in this relation.
- Trino is deployed as a coordinator (`charm-function=coordinator` or `all`) with at least one catalog in its `catalog-config`.
- If Trino is behind an ingress with TLS, its `external-hostname` is configured.

```bash
juju deploy trino-k8s --config charm-function=all --trust
```

## Establish the relation

```bash
juju integrate trino-k8s:trino-catalog superset-k8s:trino-catalog
```

Trino creates per-relation credentials and grants them to Superset through a Juju secret. No manual secret creation or `juju grant-secret` is required.

If Trino runs in a different model, offer the endpoint from the Trino model:

```bash
juju offer trino-k8s:trino-catalog
juju grant <SUPERSET_MODEL> consume admin/<TRINO_MODEL>.trino-catalog
```

Then consume and integrate it from the Superset model:

```bash
juju consume <CONTROLLER>:admin/<TRINO_MODEL>.trino-catalog
juju integrate superset-k8s trino-catalog
```

## Verify

In the Superset UI, go to **Settings** > **Database connections**. Each Trino catalog appears as a connection named after the catalog, with the catalog name in parentheses, for example `Pgsql (pgsql)`. Each connection is created with the Trino connection URL, the relation credentials, and the `database_access` permission granted to the role named by the `self-registration-role` configuration option (`Public` by default).

To follow the reconciliation in the charm log:

```bash
juju debug-log --include superset-k8s
```

## Add or remove catalogs

When catalogs are added to Trino's `catalog-config`, Superset picks them up on the next reconciliation and creates the matching connections. Removing a catalog from Trino, or removing the relation entirely, leaves the Superset connections in place. The charm never deletes connections; remove them in the UI if you want them gone.

## Understand the reconciliation model

The charm applies a partial reconciliation on every relevant event and on `update-status`:

| Change | Charm behavior |
|---|---|
| New catalog | Creates a database connection and grants `database_access` to the target role. |
| Existing catalog | Updates the connection URI when the Trino host, port, or username changes. |
| Rotated credentials | Updates the connection URI of every affected connection on the `secret-changed` event. |
| Removed catalog | Leaves the connection untouched. |
| Removed relation | Leaves all connections untouched. |

Everything else about a connection, such as its display name, extra settings, and additional permissions, is set once at creation and never overwritten, so administrator customizations survive.

Only the leader unit of a UI application reconciles; worker and beat units skip it and pick up connection details from the shared metadata database.

## Run asynchronous queries against Trino

Long-running Trino queries are best executed asynchronously so the UI stays responsive. This requires a worker application and the `GLOBAL_ASYNC_QUERIES` feature flag; see {ref}`Scale and tune performance <how-to-superset-scale-and-tune-performance>`.

## Rewrite Ranger permission-denied errors

When Trino is governed by [Ranger](https://charmhub.io/ranger-k8s), a user who opens a dashboard backed by data they are not authorized for gets a raw Trino error, such as `Access Denied: Cannot select from columns [...] in table or view users`. The charm rewrites these into a readable message and can point users at wherever your organization handles access requests:

```bash
juju config superset-k8s data-access-request-url=https://<ACCESS_REQUEST_URL>
```

The user then sees, for example:

```text
Access to catalog 'pgdata' is restricted.

Request access:
 https://data-access.example.com/request
```

When `data-access-request-url` is unset, the message asks the user to contact their administrator instead. The rewrite covers both synchronous SQL Lab errors and the asynchronous event channel used by dashboard charts.

## Troubleshoot

**Connections do not appear.** Confirm the relation is established with `juju status --relations`, then check both charm logs:

```bash
juju debug-log --include trino-k8s
juju debug-log --include superset-k8s
```

**Queries fail with permission denied.** Superset queries Trino as its own service account (`app-<APPLICATION_NAME>-<RELATION_ID>`) and impersonates the logged-in user. The account and the impersonation and query-ID permissions must exist in Ranger; see {ref}`Govern access with Ranger <how-to-govern-access>`.
