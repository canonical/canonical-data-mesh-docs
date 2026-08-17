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

```{note}
This guide uses self-signed certificates, which suit a local or test deployment. For production, obtain certificates from an ACME provider such as [Lego](https://charmhub.io/lego) and integrate it with `nginx-ingress-integrator` over the `certificates` interface instead of creating the Kubernetes secret manually.
```

## Set up TLS

Google OAuth requires the redirect URL to use HTTPS, so terminate TLS at the ingress. You can use a self-signed or production-grade TLS certificate stored in a Kubernetes secret.

For self-signed certificates, do the following:

1. Generate a private key with `openssl`, then a certificate signing request using that key. Replace `<YOUR_HOSTNAME>` with an appropriate hostname such as `airbyte-k8s.com`:

   ```bash
   openssl genrsa -out server.key 2048
   openssl req -new -key server.key -out server.csr -subj "/CN=<YOUR_HOSTNAME>"
   ```

2. Sign the signing request to create your self-signed certificate:

   ```bash
   openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt -extfile <(printf "subjectAltName=DNS:<YOUR_HOSTNAME>")
   ```

3. Add this certificate and key as a Kubernetes secret for the ingress to use:

   ```bash
   kubectl create secret tls airbyte-tls --cert=server.crt --key=server.key
   ```

   ```{note}
   If you already have a production-grade certificate, skip the certificate generation (steps 1 and 2) and go straight to this step, where you add it as a Kubernetes secret.
   ```

4. Configure the ingress provider with the Kubernetes secret and the hostname from the certificate:

   ```bash
   juju config nginx-ingress-integrator tls-secret-name=airbyte-tls service-hostname=<YOUR_HOSTNAME>
   ```

## Deploy OAuth2 Proxy

Deploy the OAuth2 Proxy charm, pinned to revision 4:

```bash
juju deploy oauth2-proxy-k8s --channel latest/edge --revision 4
```

```{note}
Stay on revision 4. Later revisions re-architect the charm around the [Canonical Identity Platform](https://charmhub.io/topics/canonical-identity-platform): they drop the `nginx-route` interface which this guide relates over, drop the `authenticated-emails-list` allowlist, and move authentication to an `oauth`/IdP relation. Airbyte supports only basic authentication, not OIDC, so it depends on OAuth2 Proxy for Google authentication and on the allowlist to restrict who can sign in. Bumping the revision will break the deployment.
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

Validate that your ingress has been created with the TLS certificate:

```bash
kubectl get ingress
kubectl describe <YOUR_INGRESS_NAME>
```

The `describe` command shows something similar to the following, with the Kubernetes secret you configured under `TLS`:

```text
Name:             relation-201-airbyte-k8s-com-ingress
Labels:           app.juju.is/created-by=nginx-ingress-integrator
                  nginx-ingress-integrator.charm.juju.is/managed-by=nginx-ingress-integrator
Namespace:        airbyte-model
Address:          <list-of-ips>
Ingress Class:    nginx-ingress-controller
Default backend:  <default>
TLS:
  airbyte-tls terminates airbyte-k8s.com
```

Open the external hostname in a browser. Airbyte now prompts for Google sign-in, and only the accounts in `authenticated-emails-list` can reach it.
