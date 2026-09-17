(how-to-superset-expose-with-ingress)=

# Expose Superset with ingress

This guide describes how to serve the Superset UI on a hostname of your own and terminate TLS in front of it.

Superset requires an ingress for anything beyond local experimentation: single sign-on, alerts and reports links, and browser access from outside the cluster all depend on a stable external URL.

Superset requires the standard `ingress` interface, so any charm that provides it will do. The provider decides the external URL and reports it back to Superset. Two providers are covered here: [traefik-k8s](https://charmhub.io/traefik-k8s), which is self-contained, and the [Nginx Ingress Integrator](https://charmhub.io/nginx-ingress-integrator), which reuses an existing cluster ingress controller.

Superset serves its UI at the root path. Both providers path-prefix by default, placing the UI under `/<MODEL>-<APPLICATION>`, where its assets and OIDC callback do not work. Each section below configures the provider to serve at a root URL instead.

## Using traefik-k8s

Deploy traefik in subdomain mode, so each application is served at the root of its own hostname:

```bash
juju deploy traefik-k8s --trust \
  --config routing_mode=subdomain \
  --config external_hostname=<YOUR_DOMAIN>
```

Integrate it with Superset:

```bash
juju integrate superset-k8s:ingress traefik-k8s:ingress
```

Traefik publishes an `https://` URL only once it holds a certificate, and Superset needs one for single sign-on. For testing, use [self-signed certificates](https://charmhub.io/self-signed-certificates):

```bash
juju deploy self-signed-certificates --channel 1/stable
juju integrate traefik-k8s:certificates self-signed-certificates:certificates
```

Outside of testing, use a production certificate provider such as [Lego](https://charmhub.io/lego) in its place.

Traefik serves each application at `<MODEL>-<APPLICATION>.<YOUR_DOMAIN>`, so Superset lands on `https://<MODEL>-superset-k8s.<YOUR_DOMAIN>`. Check the URL traefik actually published:

```bash
juju run traefik-k8s/leader show-external-endpoints
```

## Using the Nginx Ingress Integrator

This provider creates a Kubernetes ingress resource against a controller already running in the cluster. On MicroK8s, enable the built-in one:

```bash
sudo microk8s enable ingress
```

Deploy the integrator with the hostname clients use to reach Superset, and configure it to serve at the root:

```bash
juju deploy nginx-ingress-integrator --trust \
  --config service-hostname=<YOUR_HOSTNAME> \
  --config path-routes=/ \
  --config rewrite-enabled=false
```

Integrating the two charms creates the ingress resource:

```bash
juju integrate superset-k8s:ingress nginx-ingress-integrator:ingress
```

The scheme of the URL the integrator publishes is driven by its own `certificates` relation, not by any Superset config. Relate it to a certificate provider so Superset receives an `https://` URL:

```bash
juju integrate nginx-ingress-integrator:certificates lego:certificates
```

## Verify

With traefik-k8s, the URL returned by `show-external-endpoints` is the one Superset received. With the Nginx Ingress Integrator, list the ingress resource it created in the model's namespace:

```bash
kubectl get ingress -n <MODEL_NAME>
kubectl describe ingress <INGRESS_NAME> -n <MODEL_NAME>
```

Superset is now reachable at the external URL its provider published. Make sure that hostname resolves to the ingress address, either through DNS or a local `/etc/hosts` entry. With traefik in subdomain mode, that means a record for `<MODEL>-superset-k8s.<YOUR_DOMAIN>` or a wildcard record for the domain.

Relate the UI application only. The worker and beat applications serve no web traffic, and the worker learns the URL for report emails from its `external-url` option rather than from an ingress.

## Next steps

With an HTTPS URL in place, you can {ref}`enable single sign-on <how-to-superset-enable-sso>` and set `external-url` on the worker for {ref}`alerts and reports <how-to-superset-enable-alerts-and-reports>`.
