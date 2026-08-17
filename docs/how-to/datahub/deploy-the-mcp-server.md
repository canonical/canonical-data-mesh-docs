(how-to-datahub-deploy-the-mcp-server)=

# Deploy the MCP server

This guide describes how to extend a DataHub deployment with the [DataHub MCP server](https://charmhub.io/datahub-mcp-k8s), which exposes the catalog to LLM agents over the [Model Context Protocol](https://modelcontextprotocol.io/). Agents can then search datasets, walk lineage, read schemas and glossary terms, and inspect the queries that run against a dataset, grounded in the catalog rather than in the model's training data.

The MCP server is an optional companion to DataHub, deployed as its own application and related to `datahub-k8s`. Nothing else in the deployment changes when it is added or removed.

## Prerequisites

- A running DataHub deployment (see the {ref}`tutorial <tutorial-datahub-index>`)
- Juju 3.4 or later
- To authenticate callers: an ingress provider and an identity provider, as for the DataHub frontend. See {ref}`Expose DataHub with ingress <how-to-datahub-expose-with-ingress>` and {ref}`Enable single sign-on <how-to-datahub-enable-sso>`.

## Deploy and relate to DataHub

```bash
juju deploy datahub-mcp-k8s --channel latest/edge
juju integrate datahub-mcp-k8s:datahub-client datahub-k8s:datahub-client
```

The MCP server stays `blocked` until DataHub publishes the access token, then goes `active`.

On this relation, the DataHub charm creates a service account dedicated to the relation, mints a non-expiring access token for it, and passes the token in a Juju secret together with the GMS URL. The token never appears in relation data or in the charm's configuration, and removing the relation deletes the service account, which invalidates the token. See {ref}`Integrations <reference-datahub-integrations>` for the naming convention of the charm-managed resources.

The service account holds no privileges of its own; it inherits the DataHub default all-users policies, which grant metadata read. If your deployment has narrowed those policies, grant the service account a read-only metadata policy in DataHub, otherwise the tools return no results.

## Expose the server

```{important}
Without an `oauth` relation the server does not authenticate its callers: anyone who can reach the port can call the tools. Only publish it that way on a network where that is acceptable.
```

The MCP server requests ingress over the standard `ingress` interface, so any provider works, for example [Traefik](https://charmhub.io/traefik-k8s) or the [Nginx Ingress Integrator](https://charmhub.io/nginx-ingress-integrator).

Give it a hostname of its own and serve it at the root of that hostname. Clients discover where to authenticate through documents the server publishes under `/.well-known/`, which live at the root of the host: behind a path prefix, a client is told to authenticate at a URL it cannot fetch. This guide uses the Nginx Ingress Integrator, which serves at the root by default:

```bash
juju deploy nginx-ingress-integrator --channel latest/stable --trust nginx-ingress-integrator-mcp
juju config nginx-ingress-integrator-mcp \
  service-hostname=mcp.example.com path-routes=/ rewrite-enabled=false
juju integrate datahub-mcp-k8s:ingress nginx-ingress-integrator-mcp
```

The ingress must terminate TLS. An OAuth issuer identifier cannot be an `http://` URL, so the charm blocks rather than start the workload on a plain HTTP URL. Integrate the ingress with a certificates provider, for example [Lego](https://charmhub.io/lego) for an ACME-issued certificate, or [self-signed-certificates](https://charmhub.io/self-signed-certificates) in a test environment.

Point DNS for the chosen hostname at the ingress load balancer address.

## Authenticate callers

Relate an identity provider to close the endpoint:

```bash
juju integrate datahub-mcp-k8s:oauth <identity provider>
```

The server stays `blocked`, with the workload stopped, until the provider has registered the client. It does not serve a public endpoint that it cannot yet authenticate.

How callers obtain credentials depends on the provider.

### Option A: Providers that register clients themselves

Against a provider that publishes a `registration_endpoint` (Hydra, and therefore the [Canonical Identity Platform](https://charmhub.io/topics/canonical-identity-platform)) the MCP server is a plain OAuth 2.1 resource server. Clients register themselves, and the server only verifies the tokens they arrive with:

```bash
juju integrate datahub-mcp-k8s:oauth hydra:oauth
```

Nothing further is needed. Users pass their client the URL and it configures itself.

### Option B: Google

Google publishes no registration endpoint, so a client pointed at it has no way to obtain credentials of its own. The charm therefore runs an OAuth proxy in front of Google: an authorization server of its own that registers callers on demand and holds the single Google client this deployment owns. Callers still pass their client nothing but the URL, and Google only ever redirects to the server's own fixed callback rather than to whichever loopback port a client happens to listen on.

The deployment needs one Google client of its own, separate from the one DataHub's SSO uses, because a client's redirect URIs are its own:

1. In the [Google Cloud console](https://console.cloud.google.com/apis/credentials), create an OAuth client of type **Web application**. A desktop client is not needed: the secret stays on the server and never reaches users.
2. Under **Authorized redirect URIs**, add exactly one entry, `https://<your-hostname>/auth/callback`, for example `https://mcp.example.com/auth/callback`.
3. Deploy a second integrator carrying that client, with the same endpoints as DataHub's (see {ref}`Enable single sign-on <how-to-datahub-enable-sso>` for the configuration file), and relate it:

```bash
juju deploy oauth-external-idp-integrator --config mcp-idp-config.yaml oauth-external-idp-integrator-mcp
juju integrate datahub-mcp-k8s:oauth oauth-external-idp-integrator-mcp:oauth
```

Two consequences of the proxy are worth knowing:

- **The workload needs egress to `oauth2.googleapis.com`**, because it exchanges the authorization code and validates tokens server-side. Behind a filtering proxy, allow that host. The charm forwards the model's `juju-http-proxy`, `juju-https-proxy`, and `juju-no-proxy` settings to the workload; see {ref}`Configure model proxies <how-to-datahub-configure-model-proxies>`.
- **Run a single unit.** The proxy is the authorization server, and it holds its client registrations and the tokens it issues in the unit, so a second unit does not recognize the first one's tokens. Providers that register clients themselves keep no such state and scale normally.

## Enable the mutation tools

By default, the server exposes the read-only tool set. To also expose the tools that write tags, glossary terms, owners, domains, and descriptions:

```bash
juju config datahub-mcp-k8s enable-mutation-tools=true
```

Every write is attributed to the shared service account rather than to the user whose agent made the call, and the service account holds no write privileges by default, so this also requires granting it a write policy in DataHub. See {ref}`MCP server <reference-datahub-mcp-server>` for the full tool set.

## Verify

Check that the application is active and note the URL the ingress published:

```bash
juju status --relations
```

## Connect a client

The URL is all a client needs, including when the endpoint authenticates its callers. It discovers where to authenticate and obtains its own credentials on its own; no client ID, secret, or callback port belongs in a client's configuration:

```json
{
  "mcpServers": {
    "datahub": {
      "type": "http",
      "url": "https://mcp.example.com/mcp"
    }
  }
}
```

The first call opens a browser for the user to log in with the identity provider. What a caller sees in the catalog does not depend on who they are: every call reaches DataHub as the one service account from the `datahub-client` relation. The identity decides whether a caller may call the server at all, not what it will show them.

## Remove the MCP server

```bash
juju remove-application datahub-mcp-k8s
```

Removing the relation or the application deletes the DataHub service account and invalidates its token. The catalog and everything else in the deployment are unaffected.
