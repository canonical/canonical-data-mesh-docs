(how-to-airbyte-secure-deployments)=

# Secure Airbyte deployments

This guide describes how to add security features such as encryption and authentication to a Charmed Airbyte deployment.

## Terminate TLS at ingress

Airbyte can terminate Transport Layer Security (TLS) at the ingress using the [Nginx Ingress Integrator charm](https://charmhub.io/nginx-ingress-integrator).

Deploy it:

```bash
juju deploy nginx-ingress-integrator --trust
```

### Use Kubernetes secrets

You can use a self-signed or production-grade TLS certificate stored in a Kubernetes secret. The secret is then associated with the ingress to encrypt traffic between clients and Airbyte.

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

4. Configure the ingress provider with the Kubernetes secret and the hostname from the certificate. Airbyte exposes the standard `ingress` interface and does not terminate TLS itself, so these options are set on the `nginx-ingress-integrator` charm, which terminates TLS in front of Airbyte:

   ```bash
   juju config nginx-ingress-integrator tls-secret-name=airbyte-tls service-hostname=<YOUR_HOSTNAME>
   ```

5. Integrate Airbyte with the ingress provider over the standard `ingress` interface to create your ingress resource. The Nginx Ingress Integrator is used here, but any compatible ingress provider (such as Traefik) works the same way:

   ```bash
   juju integrate airbyte-k8s nginx-ingress-integrator
   ```

```{note}
If you have a production-grade certificate, skip to step 3.
```

Validate that your ingress has been created with the TLS certificates:

```bash
kubectl get ingress
kubectl describe <YOUR_INGRESS_NAME>
```

The ingress has the format `<relation_num>-<hostname>-ingress`. The `describe` command shows something similar to the following, with the Kubernetes secret you configured under `TLS`:

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

## Enable Google OAuth

Enabling Google OAuth for Charmed Airbyte lets users authenticate with their Google accounts, streamlining login and increasing security. Google OAuth is handled by the `oauth2-proxy-k8s` charm, which sits in front of Airbyte and is exposed through `nginx-ingress-integrator`.

### Deploy OAuth2 Proxy

Deploy the OAuth2 Proxy charm:

```bash
juju deploy oauth2-proxy-k8s --channel 0.1/stable
```

### Obtain OAuth2 credentials

If you do not already have OAuth2 credentials set up, follow these steps:

1. Navigate to the [Google Cloud console credentials page](https://console.cloud.google.com/apis/credentials).
2. Click **Create Credentials**.
3. Select **OAuth client ID**.
4. Select the application type (**Web application**).
5. Name the application.
6. Add an authorized redirect URI (`https://<host>:8088/oauth-authorized/google`).
7. Create and download your client ID and client secret.

### Apply OAuth configuration to the OAuth2 Proxy charm

The `oauth2-proxy-k8s` charm manages all OAuth configuration for Airbyte. Create a file `oauth2-proxy.yaml` containing your Google OAuth details:

```yaml
oauth2-proxy-k8s:
  client_id: "<GOOGLE_CLIENT_ID>"
  client_secret: "<GOOGLE_CLIENT_SECRET>"
  cookie_secret: "<RANDOM_32_BYTE_SECRET>"
  external_hostname: "airbyte.company.com"
  authenticated_emails_list: "user1@company.com,user2@company.com,<SERVICE_ACCOUNT>"
  additional_config: "--upstream-timeout=1200s --skip-jwt-bearer-tokens=true --extra-jwt-issuers=https://accounts.google.com=<GOOGLE_CLIENT_ID>"
  upstream: "http://airbyte-k8s:8001"
```

- `cookie_secret` must be a 32-byte base64-encoded value.
- `external_hostname` must match what Google OAuth expects.
- `authenticated_emails_list` controls who can access Airbyte.

Apply the configuration:

```bash
juju config oauth2-proxy-k8s --file=path/to/oauth2-proxy.yaml
```

### Relate OAuth2 Proxy with Nginx Ingress Integrator

Relate the OAuth2 Proxy with the Nginx Ingress Integrator to expose it through the ingress:

```bash
juju relate oauth2-proxy-k8s nginx-ingress-integrator
```

This updates the running `oauth2-proxy` unit and enforces Google OAuth in front of Airbyte.
