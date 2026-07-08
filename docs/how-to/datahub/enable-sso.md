# Enable single sign-on

This guide describes how to enable SSO login for DataHub. The charm receives OIDC credentials over the `oauth` relation, either from the [Canonical Identity Platform](https://charmhub.io/topics/canonical-identity-platform) or from an [oauth-external-idp-integrator](https://charmhub.io/oauth-external-idp-integrator) bridging an external identity provider such as Google or Azure AD.

## Prerequisites

- A running DataHub deployment
- An HTTPS frontend ingress. SSO requires the `frontend-ingress` URL to be HTTPS; the charm stays blocked with `OIDC requires an HTTPS ingress URL` until that is the case. See [Expose DataHub with ingress](expose-with-ingress.md).

## Option A: Use the Canonical Identity Platform

If you run the Canonical Identity Platform, relate DataHub directly to Hydra:

```bash
juju integrate datahub-k8s:oauth hydra:oauth
```

Hydra issues a client for DataHub and delivers the issuer URL and credentials over the relation.

## Option B: Use an external identity provider

For an external IdP, deploy the integrator charm with the IdP's credentials and endpoints. Using Google as the example:

1. Create OAuth credentials in the [Google Cloud console](https://console.cloud.google.com/apis/credentials): click **Create Credentials**, choose **OAuth client ID**, and select **Web application** as the type.
2. Under **Authorized redirect URIs**, add your DataHub frontend URL followed by `/callback/oidc`, for example `https://datahub.example.com/callback/oidc`.
3. Note the client ID and client secret.
4. Write the integrator configuration to a file:

```yaml
# idp-config.yaml
oauth-external-idp-integrator:
  issuer_url: https://accounts.google.com
  authorization_endpoint: https://accounts.google.com/o/oauth2/auth
  token_endpoint: https://oauth2.googleapis.com/token
  introspection_endpoint: https://oauth2.googleapis.com/tokeninfo
  userinfo_endpoint: https://www.googleapis.com/oauth2/v1/userinfo
  jwks_endpoint: https://www.googleapis.com/oauth2/v3/certs
  scope: "openid email profile"
  client_id: <CLIENT_ID>
  client_secret: <CLIENT_SECRET>
```

5. Deploy the integrator and relate it to DataHub:

```bash
juju deploy oauth-external-idp-integrator --config idp-config.yaml
juju integrate datahub-k8s:oauth oauth-external-idp-integrator:oauth
```

## Verify

Open the DataHub frontend URL in a browser. The login page now offers **Sign in with SSO**, which redirects to your identity provider and back to DataHub after authentication.

Users are matched by the `email` claim, so DataHub user profiles (`urn:li:corpuser:<email>`) are preserved as long as the same IdP, or one issuing the same email addresses, backs the relation.

## Large session cookies

Some identity providers issue tokens large enough that the OIDC session cookie exceeds reverse proxy header buffers. In that case, switch the frontend session store from cookies to the Play framework cache:

```bash
juju config datahub-k8s use-play-cache-session-store=true
```

Note that this option makes the frontend service stateful and is currently unsupported for horizontally scaled deployments.

## Deployments behind an HTTP proxy

If your model runs in a restricted network, the frontend needs proxy settings to reach the identity provider. See [Configure model proxies](configure-model-proxies.md).
