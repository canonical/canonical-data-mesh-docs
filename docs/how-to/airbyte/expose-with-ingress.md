(how-to-airbyte-expose-with-ingress)=

# Expose Airbyte with ingress

This guide describes how to expose Airbyte's web interface through an ingress, with TLS termination at the ingress.

Airbyte serves its web interface on port 8001 and exposes it over the standard `ingress` interface. Any charm that provides the `ingress` interface works, for example [Traefik](https://charmhub.io/traefik-k8s) or the [Nginx Ingress Integrator](https://charmhub.io/nginx-ingress-integrator). This guide uses Traefik.

```{note}
To put Google authentication in front of Airbyte instead of exposing it directly, see {ref}`Enable Google OAuth <how-to-airbyte-enable-google-oauth>`, which fronts Airbyte with OAuth2 Proxy over the Nginx Ingress Integrator.
```

## Prerequisites

- A running Airbyte deployment (see the {ref}`tutorial <tutorial-airbyte-index>`)
- A Kubernetes cluster with load balancer support. On MicroK8s, enable MetalLB with an address range that suits your network:

```bash
microk8s enable metallb:10.64.140.43-10.64.140.49
```

## Deploy the ingress charm

Deploy Traefik and relate it to Airbyte:

```bash
juju deploy traefik-k8s --channel latest/stable --trust

juju integrate airbyte-k8s:ingress traefik-k8s
```

Traefik assigns an external URL and publishes it back to Airbyte over the relation.

## Enable TLS

TLS is terminated on the Traefik side by integrating it with a certificates provider. For a local environment, use [self-signed certificates](https://charmhub.io/self-signed-certificates):

```bash
juju deploy self-signed-certificates --channel latest/stable
juju integrate traefik-k8s:certificates self-signed-certificates:certificates
```

For production, obtain certificates from an ACME provider such as [Lego](https://charmhub.io/lego) instead, and integrate it with Traefik over the `certificates` interface the same way.

## Find the published URL

Ask Traefik for the endpoints it proxies:

```bash
juju run traefik-k8s/0 show-proxied-endpoints
```

Open the returned URL in a browser to reach the Airbyte web interface over the ingress.

## Serve Airbyte at a hostname

By default, Traefik routes by path prefix (for example `https://<IP>/<MODEL>-airbyte-k8s`). Airbyte's web interface is a single-page application, so serve it at the root of a hostname rather than behind a path prefix.

- With Traefik, set a hostname and subdomain routing:

```bash
juju config traefik-k8s external_hostname=example.com routing_mode=subdomain
```

- Alternatively, use the Nginx Ingress Integrator, which serves applications at the root of a hostname by default.

Point DNS for the chosen hostname at the ingress load balancer IP.
