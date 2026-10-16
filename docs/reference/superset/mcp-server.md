(reference-superset-mcp-server)=

# MCP server

**MCP server** is the `mcp` charm function of `superset-k8s`, the [Model Context Protocol](https://modelcontextprotocol.io/) interface built into Apache Superset. It is an optional extension of a Superset deployment: it runs as a fourth application of the same charm, alongside the UI, worker, and beat applications, and shares their metadata database. See {ref}`Deploy the MCP server <how-to-superset-deploy-the-mcp-server>`.

## Workload

| Property | Value |
|---|---|
| Container | `superset`, the same container every charm function uses |
| Port | 5008 |
| Transport | Streamable HTTP, stateless by default (no MCP session between requests) |
| Juju | 3.4 or later |

The `external-url` configuration option must name the public URL of the Superset UI application. Tools that generate links back into Superset build them from it; without it, they fall back to an unreachable `http://localhost:5008`.

### HTTP routes

| Route | Authentication | Description |
|---|---|---|
| `/mcp` | Bearer token, when an identity provider is related | The MCP endpoint. `POST` only; `GET` returns `405`. |
| `/health` | None | Health route, also used by the Pebble check. |
| `/.well-known/oauth-protected-resource/mcp` | None | [RFC 9728](https://datatracker.ietf.org/doc/html/rfc9728) resource metadata, naming the authorization server. Served only when the charm fronts Google as an OAuth proxy. |
| `/.well-known/oauth-authorization-server` | None | [RFC 8414](https://datatracker.ietf.org/doc/html/rfc8414) metadata, served only when the charm fronts Google as an OAuth proxy. |
| `/authorize` | None | Authorization endpoint of the OAuth proxy, served only when the charm fronts Google. Where a caller sends the user to sign in. |
| `/token` | Client credentials | Token endpoint of the OAuth proxy, served only when the charm fronts Google. Exchanges an authorization code, and refreshes. |
| `/register` | None | [RFC 7591](https://datatracker.ietf.org/doc/html/rfc7591) dynamic client registration, served only when the charm fronts Google and `mcp-auth-client-registration` is left on. |
| `/auth/callback` | None | Redirect URI of the OAuth proxy, served only when the charm fronts Google. Register it on the Google client, alongside the redirect URI of any client configured against Google directly. |

The `.well-known` documents are served at the root of the host, so the server needs a hostname of its own rather than a path on another one.

## Relation endpoints

The `mcp` application uses the same `postgresql_db`, `redis`, `ingress`, and `oauth` endpoints documented in {ref}`Integrations <reference-superset-integrations>` — it is not a separate relation type. It has no use for `smtp` or `trino-catalog`: no tool sends mail, and Trino-backed connections created through the UI application's own relation are already visible in the shared metadata database. `certificates` is still relevant if `mcp` needs to trust a private CA.

The application stays `blocked` until it has exactly one identity source: the `oauth` relation or the `mcp-dev-username` configuration option.

## Tools

| Tool | Description |
|---|---|
| `list_charts` | List chart metadata: ID, name, visualization type, URL, and last modified time. |
| `get_chart_info` | Fetch a saved or unsaved chart's configuration and metadata by ID or form-data key. |
| `get_chart_data` | Return the data behind a chart, for analysis without rendering an image. |
| `get_chart_preview` | Render a chart preview as a URL, an ASCII chart, a table, or Vega-Lite content. |
| `get_chart_type_schema` | Return the fields, types, and constraints for a chart type, to use before calling `generate_chart` or `update_chart`. |
| `generate_chart` | Build a chart from a dataset and a visualization type, previewing it by default. |
| `update_chart` | Modify an existing saved chart. |
| `update_chart_preview` | Modify the cached, unsaved state of a chart previously built with `generate_chart`. |
| `list_dashboards` | List dashboard metadata: title, slug, URL, and last modified time. |
| `get_dashboard_info` | Fetch a dashboard's title, charts, and layout by ID, UUID, or slug. |
| `generate_dashboard` | Create a new dashboard from a set of charts. |
| `add_chart_to_existing_dashboard` | Add a chart to an existing dashboard, auto-positioned in a two-column grid. |
| `list_databases` | List connected databases: name, backend type, and permissions. |
| `get_database_info` | Return a database connection's configuration and capabilities. |
| `list_datasets` | List dataset metadata: table name, schema, and last modified time. |
| `get_dataset_info` | Return the columns, metrics, and schema details of a dataset. |
| `create_virtual_dataset` | Create a dataset from a SQL query, for data that does not map to a single table. |
| `generate_explore_link` | Build a link to the Explore page for a chart configuration, the preferred way to hand a visualization to a user. |
| `execute_sql` | Run a SQL query against a connected database through SQL Lab. |
| `open_sql_lab_with_context` | Build a URL that opens SQL Lab with a query pre-filled. |
| `save_sql_query` | Save a SQL query so a user can reload, share, or find it on the Saved Queries page. |
| `get_instance_info` | Return counts, activity metrics, and database types for the deployment. |
| `get_schema` | Return the schema metadata needed to construct valid chart, dataset, dashboard, or database queries. |
| `health_check` | Confirm the MCP service is reachable. Takes no parameters. |

`mcp-disabled-tools` removes tools from this set by name; unknown names are logged and otherwise ignored.

There is no separate toggle for write tools: every tool above is served, and each one checks the calling user's own Superset role against the permission it needs (for example, `update_chart` requires write access to charts) before it runs. Under `mcp-dev-username`, that check is skipped entirely, so every tool succeeds regardless of role.

## Identity resolution

The role check above needs to match a bearer token to an existing Superset user, resolved from the token's claims (`sub` or `email`, depending on the provider). This only matches an account that already exists - a first-time caller needs one created in advance, for example through the UI application's own OAuth self-registration (`self-registration-role`).

## Client registration

Dynamic, self-service client registration exists only when the charm fronts Google. Against a non-Google provider, `mcp` has no authorization or token endpoint of its own and publishes no discovery document - it only verifies tokens, reusing the same client ID and secret the UI application's own single sign-on uses, without the means to issue them.

| Identity provider | How a caller gets in |
|---|---|
| Google | By default, from the OAuth proxy this server runs. A caller already holding a token Google issued to this deployment's own client is accepted too, whether or not it went through the proxy. |
| A non-Google provider (Hydra, Canonical Identity Platform) | A client-credentials token requested directly from the provider, with an audience including the client ID (checked against the token's `aud` claim). |

That token's `sub` claim is the client ID itself, so every caller presenting it resolves to one shared Superset account, named after that client ID (visible as the `MCP_AUTH_CLIENT_ID` environment variable).

`mcp-auth-client-registration=false` withdraws `/register` and stops the Google proxy resolving any client through `/authorize` or `/token`, not even its own - only a caller already holding a token Google issued directly to this deployment's client still gets in. It is a no-op against a non-Google provider, which never served `/register` in the first place.

## Scaling

Every unit answers independently: a caller can reach any one of them.

- **Against a non-Google provider**, the server holds no state of its own: each token is validated locally against the provider's cached JWKS, with no per-call round trip to the provider and no introspection path, so a token keeps working until it expires even if the provider revokes it in the meantime.
- **Fronting Google**, the OAuth proxy stores client registrations and issued tokens in a Fernet-encrypted Redis store instead of on a pod's local disk, so every unit shares them and a caller stays signed in across a restart or a scale-out. The encryption key comes from the signing keys secret; until that secret reaches the workload, the proxy falls back to an unshared, encrypted local-disk store.

## Network egress

Fronting Google, the workload validates tokens and exchanges authorization codes server-side and needs egress to `oauth2.googleapis.com`.

## Observability

Like the other three applications, `mcp` exposes `metrics-endpoint` and `logging`. Its metrics use the `superset_mcp_*` namespace, over the same StatsD-exporter pipeline described in {ref}`Observability <reference-superset-observability>`, and the `grafana-dashboard` relation adds three panels specific to it: **MCP tool calls - actions per second (all tools)**, **MCP tool calls, grouped by category**, and **MCP errors**.

## Generated reference on Charmhub

Configuration options are generated from the charm itself and published on Charmhub, they are not duplicated here:

- [Configuration options](https://charmhub.io/superset-k8s/configurations)

## Source and issues

- [Source repository](https://github.com/canonical/superset-k8s-operator)
- [Report a bug](https://github.com/canonical/superset-k8s-operator/issues)
