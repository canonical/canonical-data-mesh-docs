(reference-datahub-mcp-server)=

# MCP server

**DataHub MCP server** ([datahub-mcp-k8s](https://charmhub.io/datahub-mcp-k8s)) is a Kubernetes operator for the [DataHub MCP server](https://pypi.org/project/mcp-server-datahub/), the [Model Context Protocol](https://modelcontextprotocol.io/) interface to the catalog. It is an optional extension of a DataHub deployment: it runs alongside `datahub-k8s` and relates to it. See {ref}`Deploy the MCP server <how-to-datahub-deploy-the-mcp-server>`.

## Workload

| Property | Value |
|---|---|
| Container | `mcp-server`, one per unit |
| Port | 8000 |
| Transport | Streamable HTTP, stateless (no MCP session between requests) |
| Workload | [`mcp-server-datahub`](https://pypi.org/project/mcp-server-datahub/), packaged as the `datahub-mcp` [rock](https://documentation.ubuntu.com/rockcraft/en/stable/) resource |
| Juju | 3.4 or later |

### HTTP routes

| Route | Authentication | Description |
|---|---|---|
| `/mcp` | Bearer token, when an identity provider is related | The MCP endpoint. `POST` only; `GET` returns `405`. |
| `/health` | None | Health route, also used by the Pebble check. |
| `/.well-known/oauth-protected-resource/mcp` | None | [RFC 9728](https://datatracker.ietf.org/doc/html/rfc9728) resource metadata, naming the authorization server. Advertised in the `WWW-Authenticate` header of a `401`. |
| `/.well-known/oauth-authorization-server` | None | [RFC 8414](https://datatracker.ietf.org/doc/html/rfc8414) metadata, served only when the charm fronts Google as an OAuth proxy. |
| `/auth/callback` | None | Redirect URI of the OAuth proxy, served only when the charm fronts Google. This is the one URI to register on the Google client. |

The `.well-known` documents are served at the root of the host, so the server needs a hostname of its own rather than a path on another one.

## Relation endpoints

### Required

| Endpoint | Interface | Description |
|---|---|---|
| `datahub-client` | `datahub_client` | GMS URL and the access token of a dedicated service account, provided by [datahub-k8s](https://charmhub.io/datahub-k8s). Single relation. |

### Optional

| Endpoint | Interface | Description |
|---|---|---|
| `ingress` | `ingress` | Publishes the endpoint at an external URL. Required for client authentication, and the URL must be HTTPS. |
| `oauth` | `oauth` | OIDC credentials used to authenticate callers, from the Canonical Identity Platform or an [oauth-external-idp-integrator](https://charmhub.io/oauth-external-idp-integrator). Without it, callers are not authenticated. |

## Tools

The read-only tool set is served by default:

| Tool | Description |
|---|---|
| `search` | Full-text search across catalog entities, with filters and sorting. |
| `get_entities` | Fetch one or more entities by URN, with their properties, ownership, tags, and glossary terms. |
| `get_lineage` | Walk upstream or downstream lineage of an entity, for a whole dataset or a single column. |
| `get_lineage_paths_between` | Return the lineage paths connecting two entities or columns, including the intermediate steps. |
| `list_schema_fields` | List the schema fields of a dataset, with keyword filtering and pagination. |
| `get_dataset_queries` | Return the SQL queries recorded against a dataset or column. |

`enable-mutation-tools=true` additionally serves the write tools: `add_tags`, `remove_tags`, `add_terms`, `remove_terms`, `add_owners`, `remove_owners`, `set_domains`, `remove_domains`, `update_description`, `add_structured_properties`, and `remove_structured_properties`. Writes are attributed to the shared service account, which holds no write privileges unless granted them in DataHub.

## Service account

The DataHub charm creates one service account per `datahub-client` relation:

| Resource | Naming pattern | Notes |
|---|---|---|
| Service account | `[juju] <app>-<relation-id>` | Created with no privileges of its own; inherits the DataHub default all-users policies, which grant metadata read. |
| Access token | Non-expiring, passed in a Juju secret granted to the relation | Never written to relation data, to charm configuration, or to the logs. Deleted with the service account when the relation is removed. |

## Scaling

| Identity provider | Units |
|---|---|
| None, or one that registers clients itself (Hydra, Canonical Identity Platform) | Scalable. The server holds no state; tokens are checked against the provider. |
| Google | One. The charm runs an OAuth proxy that is itself the authorization server, holding client registrations and issued tokens in the unit. |

## Network egress

Fronting Google, the workload validates tokens and exchanges authorization codes server-side and needs egress to `oauth2.googleapis.com`. The charm forwards the model's `juju-http-proxy`, `juju-https-proxy`, and `juju-no-proxy` settings to the workload; see {ref}`Configure model proxies <how-to-datahub-configure-model-proxies>`.

## Generated reference on Charmhub

Configuration options, actions, and relation endpoints are generated from the charm itself and published on Charmhub, they are not duplicated here:

- [Configuration options](https://charmhub.io/datahub-mcp-k8s/configurations)
- [Integrations](https://charmhub.io/datahub-mcp-k8s/integrations)

## Source and issues

- [Source repository](https://github.com/canonical/datahub-mcp-k8s-operator)
- [Report a bug](https://github.com/canonical/datahub-mcp-k8s-operator/issues)
