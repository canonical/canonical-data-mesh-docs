(how-to-datahub-expose-with-ingress)=

# Expose DataHub with ingress

This guide describes how to expose DataHub's web frontend and metadata service (GMS) API through an ingress, with TLS termination at the ingress.

DataHub exposes two independent ingress endpoints over the standard `ingress` interface:

- `frontend-ingress` - the web UI on port 9002. This is what users open in a browser and what SSO redirects flow through.
- `gms-ingress` - the GMS API on port 8080, used by external ingestion jobs, the DataHub CLI, and API clients.

Any charm that provides the `ingress` interface works, for example [Traefik](https://charmhub.io/traefik-k8s) or the [Nginx Ingress Integrator](https://charmhub.io/nginx-ingress-integrator). This guide uses Traefik.

## Prerequisites

- A running DataHub deployment (see the {ref}`tutorial <tutorial-datahub-index>`)
- A Kubernetes cluster with load balancer support. On MicroK8s, enable MetalLB with an address range that suits your network:

```bash
microk8s enable metallb:10.64.140.43-10.64.140.49
```

## Deploy the ingress charms

Deploy one Traefik instance per endpoint:

```bash
juju deploy traefik-k8s --channel latest/stable --trust traefik-frontend
juju deploy traefik-k8s --channel latest/stable --trust traefik-gms

juju integrate datahub-k8s:frontend-ingress traefik-frontend
juju integrate datahub-k8s:gms-ingress traefik-gms
```

Each Traefik assigns an external URL and publishes it back to DataHub over the relation. The charm derives its OIDC base URL from the `frontend-ingress` URL automatically, so no manual hostname configuration is needed.

## Enable TLS

TLS is terminated on the Traefik side by integrating it with a certificates provider. For a local environment, use [self-signed certificates](https://charmhub.io/self-signed-certificates):

```bash
juju deploy self-signed-certificates --channel latest/stable
juju integrate traefik-frontend:certificates self-signed-certificates:certificates
juju integrate traefik-gms:certificates self-signed-certificates:certificates
```

For production, use an ACME provider such as [Lego](https://charmhub.io/lego) instead. Single sign-on requires the frontend URL to be HTTPS - see {ref}`Enable single sign-on <how-to-datahub-enable-sso>`.

## Find the published URLs

Ask each Traefik for the endpoints it proxies:

```bash
juju run traefik-frontend/0 show-proxied-endpoints
juju run traefik-gms/0 show-proxied-endpoints
```

Verify the GMS API through its ingress:

```bash
curl -k <GMS_URL>/config
```

This returns a JSON document that includes the running DataHub version.

## Use host-based routing for the frontend

By default, Traefik routes by path prefix (for example `https://<IP>/<MODEL>-datahub-k8s`). The DataHub frontend is a single-page application compiled with absolute asset paths, so it does not render correctly behind a path prefix: the page loads, but the browser requests `/assets/...` without the prefix and gets a blank page.

For the frontend, use host-based routing so DataHub is served at the root of a hostname:

- With Traefik, set a hostname and subdomain routing:

```bash
juju config traefik-frontend external_hostname=example.com routing_mode=subdomain
```

- Alternatively, use the Nginx Ingress Integrator, which serves applications at the root of a hostname by default.

Point DNS for the chosen hostname at the ingress load balancer IP. Path-prefix routing remains fine for the GMS API endpoint, which is not a browser application.
