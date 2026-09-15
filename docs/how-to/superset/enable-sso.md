(how-to-superset-enable-sso)=

# Enable single sign-on

This guide describes how to let users log in to Superset with an OIDC identity provider instead of local Superset credentials, and how to control the role they receive on first login.

Superset takes the provider's endpoints and its own client credentials from the `oauth` relation, which any charm providing the `oauth` interface can serve. This guide uses [oauth-external-idp-integrator](https://charmhub.io/oauth-external-idp-integrator), which passes on the details of a client you register with the identity provider yourself.

## Prerequisites

- Superset is served over HTTPS on a stable hostname. See {ref}`Expose Superset with ingress <how-to-superset-expose-with-ingress>`. This is required, not advisory: the charm builds its redirect URI from the external URL the ingress provider reports, and blocks with `OAuth requires an HTTPS ingress URL` if an `oauth` relation exists without one.
- An OIDC identity provider on which you can register a client application.

## Register a client with the identity provider

Register Superset with the identity provider as a web application that uses the authorization code flow, with this redirect URI:

```text
https://<YOUR_HOSTNAME>/oauth-authorized/oidc
```

Note the client ID and client secret the provider issues.

```{important}
The callback path is `/oauth-authorized/oidc` whichever identity provider is
behind it, because the charm registers the provider under the name `oidc`. A
mismatch here fails the login with `redirect_uri_mismatch`.
```

Then collect the provider's endpoints. An OIDC provider lists them in its discovery document at `<ISSUER_URL>/.well-known/openid-configuration`.

## Relate the identity provider

Write the integrator configuration to a file, so that the client secret does not end up in your shell history:

```yaml
# idp-config.yaml
oauth-external-idp-integrator:
  issuer_url: <ISSUER_URL>
  authorization_endpoint: <AUTHORIZATION_ENDPOINT>
  token_endpoint: <TOKEN_ENDPOINT>
  introspection_endpoint: <INTROSPECTION_ENDPOINT>
  userinfo_endpoint: <USERINFO_ENDPOINT>
  jwks_endpoint: <JWKS_ENDPOINT>
  scope: "openid email profile"
  client_id: <CLIENT_ID>
  client_secret: <CLIENT_SECRET>
```

Deploy the integrator and relate it to Superset:

```bash
juju deploy oauth-external-idp-integrator --channel latest/edge --config idp-config.yaml
juju integrate superset-k8s:oauth oauth-external-idp-integrator:oauth
```

Relate the UI application only. It is the one that serves logins; workers and the beat scheduler do not authenticate users.

## What the charm exchanges

On relating, the charm publishes its client registration and reads the provider's answer back:

| Published by Superset | Value |
|---|---|
| `redirect_uri` | `<external URL>/oauth-authorized/oidc` |
| `scope` | `openid email profile` |
| `grant_types` | `authorization_code` |

The provider answers with the issuer URL, the authorization, token, user info, and JWKS endpoints, and the client ID and secret. Superset switches to OAuth authentication only once all of them are present, so until the handshake completes it keeps serving local logins with the `admin` account. Verify the exchange with:

```bash
juju status --relations
```

## Choose the self-registration role

A Superset account is created automatically the first time a user authenticates. By default the account receives Superset's least privileged role, `Public`, and an administrator can elevate it afterwards in the UI or through the API.

To grant a different role on self-registration:

```bash
juju config superset-k8s self-registration-role=Gamma
```

The role must already exist in Superset and the value is case-sensitive; the charm validates it against the roles in the metadata database and blocks on an unknown value.

```{note}
Roles created by the {ref}`Trino integration <how-to-superset-integrate-with-trino>`
matter here: database access permissions for Trino catalogs are granted to the
role named by `self-registration-role`, so every self-registered user inherits
access to the catalogs Superset manages.
```

## Verify

Open Superset in a private browser window. You are redirected to the identity provider, and after authenticating you land in Superset as that account. Check the created account and its role under **Settings** > **List users**.

## Remove single sign-on

Removing the relation returns Superset to local authentication:

```bash
juju remove-relation superset-k8s:oauth oauth-external-idp-integrator:oauth
```

The charm drops the provider settings from the workload configuration and restarts it. Accounts created through SSO remain in the metadata database.
