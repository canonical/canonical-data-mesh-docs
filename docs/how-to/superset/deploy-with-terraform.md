(how-to-superset-deploy-with-terraform)=

# Deploy with Terraform

This guide describes how to deploy Superset with the Terraform modules shipped in the [charm repository](https://github.com/canonical/superset-k8s-operator/tree/main/terraform), using the [Terraform Juju provider](https://registry.terraform.io/providers/juju/juju/latest).

Two modules are available:

- [`terraform/charm`](https://github.com/canonical/superset-k8s-operator/tree/main/terraform/charm) - a **charm module** that deploys one `superset-k8s` application. A deployment is three applications of the same charm, so use it as a building block in your own Terraform solution by instantiating it once per `charm-function`, all three naming the same signing keys secret.
- [`terraform/product`](https://github.com/canonical/superset-k8s-operator/tree/main/terraform/product) - a **product module** that deploys the full solution into one Kubernetes model in one `terraform apply`: the UI, worker, and beat applications, PostgreSQL and Redis, Traefik with TLS from self-signed certificates, and the signing keys secret. It can optionally deploy an OAuth external IdP integrator for SSO and an SMTP integrator for alerts and reports, and relate the UI to an existing Trino catalog offer.

## Prerequisites

Every charm in the solution is a Kubernetes charm, so the product module needs a single Kubernetes model. It consumes an existing model rather than creating one; create it first, for example on MicroK8s:

```bash
juju add-model superset microk8s --config workload-storage=microk8s-hostpath
```

Pass the model's UUID, shown by `juju show-model superset`, as the `model_uuid` input. Destroying the module then leaves the model in place.

The modules need Terraform 1.14 or later and version 2 of the Juju provider.

## Set the external hostname

The module deploys Traefik and relates it to the UI, but Traefik routes by path prefix by default and Superset has to be served at the root of a hostname. Set `external_hostname` to a base domain and the module configures Traefik for host-based routing (`routing_mode = "subdomain"`) instead:

```hcl
external_hostname = "example.com"
```

Traefik then serves the UI at `<MODEL>-superset-k8s-ui.example.com`. Point that name, or a wildcard record for the domain, at Traefik's load balancer address. Left empty, Traefik has no address until the cluster gives its load balancer an IP, and even then the UI does not work in a browser and SSO cannot be enabled.

See {ref}`Expose Superset with ingress <how-to-superset-expose-with-ingress>` for why Superset needs this.

## Configure the applications

The product module owns the two options that tie the three applications together, `charm-function` and `signing-keys-secret-id`. Setting either of them in a configuration input is a validation error. Everything else is passed through one of four configuration maps:

- `superset.config` for options every application reads, such as `feature-flags`,
- `superset_ui.config`, `superset_worker.config`, and `superset_beat.config` for options only one of them reads, such as `load-examples` on the UI, `external-url` on the worker, or `log-retention-days` on the beat scheduler.

Each option's description on the [Charmhub configuration page](https://charmhub.io/superset-k8s/configurations) names the applications that read it. The module sets `server-alias` on the worker to the name of the UI application, and always deploys the beat scheduler as a single unit.

The Superset channel defaults to `latest/edge`; set `superset.channel` to deploy from a stable track.

## Keep the signing keys

The module generates the signing keys secret and grants it to the three applications. `secret-key` encrypts the database connection passwords stored in the metadata database, so to deploy against the database of an existing deployment, set `signing_keys.secret_key` to that deployment's key. Otherwise leave it unset.

## Enable SSO (optional)

Set the `oauth_external_idp_integrator_config` input with at least `client_id` and `client_secret`; the endpoint options default to Google. The module deploys the integrator and relates it to the UI only. SSO needs an HTTPS URL at a real hostname, so set `external_hostname` as well, and register `https://<UI_HOSTNAME>/oauth-authorized/oidc` as the redirect URI with the identity provider. See {ref}`Enable single sign-on <how-to-superset-enable-sso>`.

## Enable alerts and reports (optional)

Set the `smtp_integrator_config` input with at least `host` and `smtp_sender`. The module deploys the integrator and relates it to all three applications. Add `ALERT_REPORTS` to `feature-flags` in `superset.config`, and set `external-url` in `superset_worker.config` to the URL recipients open from the emails. See {ref}`Enable alerts and reports <how-to-superset-enable-alerts-and-reports>`.

## Bring your own dependencies (optional)

PostgreSQL and Redis are deployed in the model by default. Set `database_offer_url` or `redis_offer_url` to an existing offer to consume it instead; the two are independent. Set `trino_catalog_offer_url` to relate the UI to a Trino `trino-catalog` offer. Trino is not deployed by this module.

## Apply and verify

Run the module with your model UUID and any overrides, for example:

```hcl
module "superset" {
  source = "git::https://github.com/canonical/superset-k8s-operator//terraform/product"

  model_uuid        = "<MODEL_UUID>"
  external_hostname = "example.com"

  superset = {
    channel = "6/stable"
    config  = { "feature-flags" = "DASHBOARD_RBAC" }
  }

  superset_ui     = { units = 2 }
  superset_worker = { units = 3 }
}
```

Then verify as usual:

```bash
juju run superset-k8s-ui/leader get-admin-password
juju run traefik-k8s/leader show-external-endpoints
```

The worker and beat applications report `waiting for the UI to initialise the database` until the UI has initialized the metadata database, then start on their own. The module imposes no order on the relations.

## Notes

- The generated signing keys, any IdP client credentials, and any SMTP password end up in the Terraform state. Use an encrypted or remote backend for real deployments.
- The full input reference and test instructions live in the module READMEs in the repository.
