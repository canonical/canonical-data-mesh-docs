(how-to-airbyte-enable-google-oauth)=

# Enable Google OAuth

This guide describes how to put Google authentication in front of Airbyte so that users sign in with their Google accounts.

Airbyte supports only basic authentication, not OIDC, so Google sign-in is handled by the [`oauth2-proxy-k8s`](https://charmhub.io/oauth2-proxy-k8s) charm. OAuth2 Proxy sits in front of Airbyte, terminates authentication, and is exposed through the [Nginx Ingress Integrator](https://charmhub.io/nginx-ingress-integrator) over the `nginx-route` interface.

```{note}
To expose Airbyte directly over an ingress with TLS but without authentication, see {ref}`Expose Airbyte with ingress <how-to-airbyte-expose-with-ingress>` instead.
```

## Prerequisites

- A running Airbyte deployment (see the {ref}`tutorial <tutorial-airbyte-index>`)
- A Kubernetes cluster with load balancer support. On MicroK8s, enable MetalLB with an address range that suits your network:

```bash
microk8s enable metallb:10.64.140.43-10.64.140.49
```

## Deploy the Nginx Ingress Integrator

OAuth2 Proxy is exposed through the Nginx Ingress Integrator, which also terminates TLS. Deploy it:

```bash
juju deploy nginx-ingress-integrator --trust
```

## Enable TLS

Google OAuth requires the redirect URL to use HTTPS, so terminate TLS at the ingress by integrating the Nginx Ingress Integrator with a certificates provider over the `certificates` interface. It requests a certificate for the hostname OAuth2 Proxy advertises through its `external-hostname` config. For a local environment, use [self-signed certificates](https://charmhub.io/self-signed-certificates):

```bash
juju deploy self-signed-certificates --channel latest/stable
juju integrate nginx-ingress-integrator:certificates self-signed-certificates:certificates
```

For production, use an ACME provider such as [Lego](https://charmhub.io/lego) instead.

## Deploy OAuth2 Proxy

Deploy the OAuth2 Proxy charm, pinned to revision 4:

```bash
juju deploy oauth2-proxy-k8s --channel latest/edge --revision 4
```

```{note}
Stay on revision 4. Later revisions re-architect the charm around the [Canonical Identity Platform](https://charmhub.io/topics/canonical-identity-platform): they drop the `nginx-route` interface which this guide relates over, drop the `authenticated-emails-list` allowlist that restricts who can sign in, and move authentication to an `oauth`/IdP relation that Airbyte cannot use. Bumping the revision will break the deployment.
```

## Obtain OAuth2 credentials

If you do not already have OAuth2 credentials set up, follow these steps:

1. Navigate to the [Google Cloud console credentials page](https://console.cloud.google.com/apis/credentials).
2. Click **Create Credentials**.
3. Select **OAuth client ID**.
4. Select the application type (**Web application**).
5. Name the application.
6. Add an authorized redirect URI (`https://<host>:8088/oauth-authorized/google`).
7. Create and download your client ID and client secret.

## Apply OAuth configuration to the OAuth2 Proxy charm

The `oauth2-proxy-k8s` charm manages all OAuth configuration for Airbyte. Create a file `oauth2-proxy.yaml` containing your Google OAuth details:

```yaml
oauth2-proxy-k8s:
  client-id: "<GOOGLE_CLIENT_ID>"
  client-secret: "<GOOGLE_CLIENT_SECRET>"
  cookie-secret: "<RANDOM_32_BYTE_SECRET>"
  external-hostname: "airbyte.company.com"
  authenticated-emails-list: "user1@company.com,user2@company.com,<SERVICE_ACCOUNT>"
  additional-config: "--upstream-timeout=1200s --skip-jwt-bearer-tokens=true --extra-jwt-issuers=https://accounts.google.com=<GOOGLE_CLIENT_ID>"
  upstream: "http://airbyte-k8s:8001"
```

- `cookie-secret` must be a 32-byte base64-encoded value.
- `external-hostname` must match what Google OAuth expects.
- `authenticated-emails-list` controls who can access Airbyte.

Apply the configuration:

```bash
juju config oauth2-proxy-k8s --file=path/to/oauth2-proxy.yaml
```

## Relate OAuth2 Proxy with the Nginx Ingress Integrator

Relate OAuth2 Proxy with the Nginx Ingress Integrator over the `nginx-route` interface to expose it through the ingress:

```bash
juju integrate oauth2-proxy-k8s:nginx-route nginx-ingress-integrator:nginx-route
```

This updates the running `oauth2-proxy` unit and enforces Google OAuth in front of Airbyte.

## Verify

Open the external hostname in a browser. Airbyte now prompts for Google sign-in, and only the accounts in `authenticated-emails-list` can reach it.
