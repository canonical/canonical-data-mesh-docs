(how-to-superset-deploy-the-mcp-server)=

# Deploy the MCP server

This guide describes how to extend a Superset deployment with the `mcp` charm function, which exposes Superset's own functionality to LLM agents over the [Model Context Protocol](https://modelcontextprotocol.io/), grounded in the deployment's own data rather than in the model's training data. See {ref}`MCP server <reference-superset-mcp-server>` for the full tool set.

The MCP server is the same `superset-k8s` charm deployed as a fourth application, alongside the UI, worker, and beat applications. Nothing else in the deployment changes when it is added or removed.

## Prerequisites

- A running Superset deployment (see the {ref}`tutorial <tutorial-superset-index>`)
- Juju 3.4 or later
- To authenticate callers: an ingress provider and an identity provider, as for the Superset UI. See {ref}`Expose Superset with ingress <how-to-superset-expose-with-ingress>` and {ref}`Enable single sign-on <how-to-superset-enable-sso>`.
- An existing Superset account for anyone who will call the server: `mcp` resolves an existing user from the caller's identity but never creates one, so a first-time caller needs an account created in advance, for example through the UI application's own OAuth self-registration. The same applies to the username given to `mcp-dev-username`.

## Deploy and relate to Superset

```bash
juju deploy superset-k8s --channel 6/stable --config charm-function=mcp superset-k8s-mcp

juju grant-secret superset-signing-keys superset-k8s-mcp
juju config superset-k8s-mcp signing-keys-secret-id=<SECRET_ID>
juju config superset-k8s-mcp external-url=<SUPERSET_UI_URL>
juju integrate superset-k8s-mcp postgresql-k8s
juju integrate superset-k8s-mcp redis-k8s
```

The `mcp` application reads and decrypts the same metadata database as the rest of the deployment, so it must name the same signing keys secret and integrate with the same PostgreSQL and Redis charms as the UI, worker, and beat applications. It waits with `waiting for the UI to initialise the database` if related before the UI has created the schema, and starts on its own once that schema exists.

`external-url` must point at the Superset UI application's own public URL, not at the MCP server's own endpoint being set up below. Tools that generate links back into Superset - `generate_explore_link`, `open_sql_lab_with_context`, and the URL fields on chart and dashboard results - build those links from it; without it, they fall back to an unreachable `http://localhost:5008`.

The application stays `blocked` until it has exactly one identity source: an `oauth` relation or `mcp-dev-username`. Set up one of them next.

## Expose the server

Authenticating callers needs an HTTPS URL, so expose the server before relating an identity provider. Relate the same `traefik-k8s` already serving the UI (see {ref}`Expose Superset with ingress <how-to-superset-expose-with-ingress>`): in subdomain mode, one traefik application serves any number of related applications, each at the root of its own hostname:

```bash
juju integrate superset-k8s-mcp:ingress traefik-k8s:ingress
```

That root is what the MCP server needs: clients discover where to authenticate through documents it publishes under `/.well-known/`, which only resolve at the root of a host, not behind a path prefix. Traefik's subdomain mode already serves every related application that way, so nothing further is needed here. Check the URL it published for the MCP server:

```bash
juju run traefik-k8s/leader show-external-endpoints
```

Traefik serves it at `https://<MODEL>-superset-k8s-mcp.<YOUR_DOMAIN>`. Make sure that hostname resolves to the ingress address, the same as for the UI.

## Authenticate callers

Relate an identity provider to close the endpoint:

```bash
juju integrate superset-k8s-mcp:oauth <identity provider>
```

### Option A: a non-Google provider

Against a non-Google provider (Hydra, and therefore the [Canonical Identity Platform](https://charmhub.io/topics/canonical-identity-platform)), the MCP server is a plain JWT verifier: it checks the signature, issuer, and audience of a bearer token, and nothing more.

```bash
juju integrate superset-k8s-mcp:oauth hydra:oauth
```

Unlike the Google path below, this one runs no proxy of its own and publishes no discovery document: there is no `/register`, no `/.well-known/` metadata, and no authorization or token endpoint wired to it, so no browser-based per-user login is possible through `mcp` itself. It reuses the same Hydra client the UI application's own single sign-on uses - same client ID and secret - but only to verify tokens, not to issue them.

Request a token for that client directly from Hydra, using the client-credentials grant, with an audience that includes the client ID (`mcp` checks the token's `aud` claim against it). The resulting token's `sub` claim is the client ID itself, so every caller presenting it resolves to the same Superset account - create one Superset user named after that client ID (visible as `MCP_AUTH_CLIENT_ID` in the application's environment) and give it whatever role every caller on this path should share. Configure each caller's MCP client with that token as a static bearer header; see "Connect a client" below.

### Option B: Google

Google publishes no registration endpoint, so a client pointed at it has no way to obtain credentials of its own. The charm runs an OAuth proxy in front of Google instead: an authorization server of its own that registers callers on demand and holds the single Google client this deployment owns. Callers still pass their client nothing but the URL, and Google only ever redirects to the server's own fixed callback rather than to whichever loopback port a client happens to listen on.

The deployment needs one Google client of its own, separate from the one Superset's own SSO uses, because a client's redirect URIs are its own:

1. In the [Google Cloud console](https://console.cloud.google.com/apis/credentials), create an OAuth client of type **Web application**.
2. Under **Authorized redirect URIs**, add `https://<MODEL>-superset-k8s-mcp.<YOUR_DOMAIN>/auth/callback`, the hostname traefik published for the MCP server.
3. Deploy a second integrator carrying that client, reusing the configuration file from {ref}`Enable single sign-on <how-to-superset-enable-sso>` with its credentials replaced by this new Google client's, and its top-level key renamed from `oauth-external-idp-integrator` to `oauth-external-idp-integrator-mcp` - Juju matches that key to the application name it's deployed as, so it must match this one too - then relate it:

```bash
juju deploy oauth-external-idp-integrator --config mcp-idp-config.yaml oauth-external-idp-integrator-mcp
juju integrate superset-k8s-mcp:oauth oauth-external-idp-integrator-mcp:oauth
```

The workload needs egress to `oauth2.googleapis.com`, because it exchanges the authorization code and validates tokens server-side. Behind a filtering proxy, allow that host.

### Option C: a development bypass

Instead of an identity provider, `mcp-dev-username` authenticates every call as a fixed Superset user and skips the Superset-role check entirely, since there is no authenticated caller identity left to check a role against:

```bash
juju config superset-k8s-mcp mcp-dev-username=<SUPERSET_USERNAME>
```

It is mutually exclusive with the `oauth` relation: the application blocks with a `conflicting mcp auth configuration` message if both are set.

## Limit the endpoint to callers you registered

`mcp-auth-client-registration` only has an effect fronting Google. By default a caller obtains an OAuth client of its own on first connection, which is what lets a user point a client at the URL and nothing else:

```bash
juju config superset-k8s-mcp mcp-auth-client-registration=false
```

With it set to `false`, the proxy stops registering new clients. A caller needs this deployment's own client ID and secret, configured against Google's own endpoints directly. It is a no-op against a non-Google provider, which registers clients itself and cannot be stopped from doing so.

## Disable specific tools

To remove one or more tools from the set an agent can call:

```bash
juju config superset-k8s-mcp mcp-disabled-tools=generate_chart,update_chart
```

Unknown tool names are logged and otherwise ignored. See {ref}`MCP server <reference-superset-mcp-server>` for the full tool set.

## Verify

Check that the application is active and note the URL the ingress published:

```bash
juju status --relations
```

## Connect a client

What a client needs depends on which identity path is in place.

**Fronting Google, client registration on (the default):** the URL is all a client needs. It discovers where to authenticate and obtains its own credentials on its own, and no client ID, secret, or callback port belongs in its configuration:

```json
{
  "mcpServers": {
    "superset": {
      "type": "http",
      "url": "https://<MODEL>-superset-k8s-mcp.<YOUR_DOMAIN>/mcp"
    }
  }
}
```

The first call opens a browser for the user to log in with Google, and what they can do afterward depends on their own Superset role.

**Fronting Google, client registration off:** the client is instead configured by hand with this deployment's own Google client ID and secret, pointed at Google's endpoints directly rather than at the proxy's. The user still signs in through a normal Google login; only the client is provisioned in advance instead of registering on the fly.

**Against a non-Google provider:** there is no browser login through `mcp` at all. Configure the client with the bearer token obtained in "Authenticate callers" above, as a static header:

```json
{
  "mcpServers": {
    "superset": {
      "type": "http",
      "url": "https://<MODEL>-superset-k8s-mcp.<YOUR_DOMAIN>/mcp",
      "headers": {
        "Authorization": "Bearer <TOKEN>"
      }
    }
  }
}
```

Under `mcp-dev-username`, instead of any of the above, every call runs as that one fixed user, with no role check at all.

## Remove the MCP server

```bash
juju remove-application superset-k8s-mcp
```

The rest of the deployment is unaffected.
