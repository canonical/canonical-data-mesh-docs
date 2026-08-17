(how-to-superset-expose-with-ingress)=

# Expose Superset with ingress

This guide describes how to serve the Superset UI on a hostname of your own and terminate TLS in front of it, using the [Nginx Ingress Integrator](https://charmhub.io/nginx-ingress-integrator) charm.

Superset requires an ingress for anything beyond local experimentation: single sign-on, alerts and reports links, and browser access from outside the cluster all depend on a stable external URL.

## Prerequisites

An ingress controller must be available in the cluster. On MicroK8s, enable the built-in one:

```bash
sudo microk8s enable ingress
```

## Deploy the ingress integrator

```bash
juju deploy nginx-ingress-integrator --trust
```

## Configure the hostname

Set the hostname clients use to reach Superset. It defaults to the application name, which is rarely what you want:

```bash
juju config superset-k8s external-hostname=<YOUR_HOSTNAME>
```

## Provide a TLS certificate

The ingress terminates TLS with a certificate stored in a Kubernetes secret. If you already have a production certificate, create the secret from it and skip the certificate generation steps below.

To generate a self-signed certificate for testing, create a private key and a certificate signing request:

```bash
openssl genrsa -out server.key 2048
openssl req -new -key server.key -out server.csr -subj "/CN=<YOUR_HOSTNAME>"
```

Sign the request:

```bash
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt \
  -extfile <(printf "subjectAltName=DNS:<YOUR_HOSTNAME>")
```

Store the key and certificate as a Kubernetes TLS secret in the model's namespace:

```bash
kubectl create secret tls superset-tls --cert=server.crt --key=server.key -n <MODEL_NAME>
```

Point the charm at the secret:

```bash
juju config superset-k8s tls-secret-name=superset-tls
```

## Create the ingress

Integrating the two charms creates the Kubernetes ingress resource:

```bash
juju integrate superset-k8s nginx-ingress-integrator
```

## Verify

List the ingress resources in the model's namespace:

```bash
kubectl get ingress -n <MODEL_NAME>
kubectl describe ingress <INGRESS_NAME> -n <MODEL_NAME>
```

The ingress is named `<RELATION_ID>-<HOSTNAME>-ingress`, and the `describe` output shows the TLS secret terminating your hostname:

```text
Name:             relation-201-superset-k8s-com-ingress
Labels:           app.juju.is/created-by=nginx-ingress-integrator
                  nginx-ingress-integrator.charm.juju.is/managed-by=nginx-ingress-integrator
Namespace:        superset-model
Address:          <LIST_OF_IPS>
Ingress Class:    nginx-ingress-controller
Default backend:  <default>
TLS:
  superset-tls terminates superset-k8s.com
```

Superset is now reachable at `https://<YOUR_HOSTNAME>`. Make sure the hostname resolves to the ingress address, either through DNS or a local `/etc/hosts` entry.

## Next steps

With an HTTPS URL in place, you can {ref}`enable single sign-on <how-to-superset-enable-sso>` and set `superset-external-url` for {ref}`alerts and reports <how-to-superset-enable-alerts-and-reports>`.
