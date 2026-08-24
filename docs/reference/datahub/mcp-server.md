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
| `/authorize` | None | Authorization endpoint of the OAuth proxy, served only when the charm fronts Google. Where a caller sends the user to sign in. |
| `/token` | Client credentials | Token endpoint of the OAuth proxy, served only when the charm fronts Google. Exchanges an authorization code, and refreshes. |
| `/register` | None | [RFC 7591](https://datatracker.ietf.org/doc/html/rfc7591) dynamic client registration, served only when the charm fronts Google and `enable-client-registration` is left on. |
| `/auth/callback` | None | Redirect URI of the OAuth proxy, served only when the charm fronts Google. Register it on the Google client, alongside the redirect URI of any client configured against Google directly. |

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

## Client registration

A caller needs an OAuth client before it can authenticate a user, and there are two ways for it to hold one.

| | How the caller obtains a client | Example |
|---|---|---|
| Registers its own | Dynamically, on first connection, from whichever party registers clients | An MCP client on a developer's machine, configured with the URL alone |
| Registered in advance | An operator gives it this deployment's own client ID and secret | Gemini Enterprise, configured with Google's endpoints and those credentials |

Which endpoints a pre-registered caller is given depends on whether it discovers this server. One that reads the resource metadata finds the OAuth proxy and uses `/authorize` and `/token` here. One configured from a form does not look, and is given Google's own `https://accounts.google.com/o/oauth2/auth` and `https://oauth2.googleapis.com/token` instead. Both are accepted: the first presents a token this server minted, the second one Google minted, and each is admitted on the client it names.

`enable-client-registration=false` leaves only the second kind. What that changes depends on which party is the registrar:

| Identity provider | Registrar | Effect of turning registration off |
|---|---|---|
| Google | This server, acting as an OAuth proxy | `/register` is withdrawn and no longer advertised, and no client but this deployment's own resolves, so callers that registered earlier are cut off too. A caller holding a token Google issued is held to the same rule by the client named on it. |
| One that registers clients itself (Hydra, Canonical Identity Platform) | The provider | The provider cannot be stopped from registering, so tokens are refused instead unless their `client_id` or `azp` claim names this deployment's client. |

## Scaling

| Identity provider | Units |
|---|---|
| None, or one that registers clients itself (Hydra, Canonical Identity Platform) | Scalable. The server holds no state; tokens are checked against the provider. |
| Google | One. The charm runs an OAuth proxy that is itself the authorization server, holding client registrations and issued tokens in the unit. A restart loses them, and callers that discovered the proxy sign in again; a caller configured against Google directly holds a token no unit had to issue and is unaffected. |

## Network egress

Fronting Google, the workload validates tokens and exchanges authorization codes server-side and needs egress to `oauth2.googleapis.com`. The charm forwards the model's `juju-http-proxy`, `juju-https-proxy`, and `juju-no-proxy` settings to the workload; see {ref}`Configure model proxies <how-to-datahub-configure-model-proxies>`.

## Generated reference on Charmhub

Configuration options, actions, and relation endpoints are generated from the charm itself and published on Charmhub, they are not duplicated here:

- [Configuration options](https://charmhub.io/datahub-mcp-k8s/configurations)
- [Integrations](https://charmhub.io/datahub-mcp-k8s/integrations)

## Source and issues

- [Source repository](https://github.com/canonical/datahub-mcp-k8s-operator)
- [Report a bug](https://github.com/canonical/datahub-mcp-k8s-operator/issues)
