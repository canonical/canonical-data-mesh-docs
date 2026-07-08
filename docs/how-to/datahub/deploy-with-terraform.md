# Deploy with Terraform

This guide describes how to deploy DataHub with the Terraform modules shipped in the [charm repository](https://github.com/canonical/datahub-k8s-operator/tree/main/terraform), using the [Terraform Juju provider](https://registry.terraform.io/providers/juju/juju/latest).

Two modules are available:

- [`terraform/charm`](https://github.com/canonical/datahub-k8s-operator/tree/main/terraform/charm) - a **charm module** that deploys `datahub-k8s` alone. Use it as a building block in your own Terraform solution, supplying your own data platform, ingress, and secrets.
- [`terraform/product`](https://github.com/canonical/datahub-k8s-operator/tree/main/terraform/product) - a **product module** that deploys the full stack in one `terraform apply`: the data platform (PostgreSQL, Kafka with ZooKeeper, and OpenSearch with self-signed certificates), DataHub, two Traefik ingresses with TLS, and the encryption keys secret. It can optionally deploy the OAuth external IdP integrator for SSO.

## Prerequisites

The product module assumes a **single Juju controller with two clouds**: a machine cloud for the data platform and a Kubernetes cloud for DataHub. For example, with LXD and MicroK8s on one host:

```bash
juju bootstrap localhost lxd
microk8s config | juju add-k8s microk8s --controller lxd
```

The module consumes existing models; create them first:

```bash
juju add-model datahub-vm localhost
juju add-model datahub-k8s microk8s --config workload-storage=microk8s-hostpath
```

The machine host needs the OpenSearch kernel parameters described in the [tutorial](../../tutorials/datahub/02-environment-setup.md).

## Choose a data platform mode

The product module supports two modes, controlled by the offer URL variables:

- **Deploy the data platform (default)**: leave the `*_offer_url` inputs empty. The module deploys PostgreSQL, Kafka, and OpenSearch in the machine model, creates cross-model offers, and consumes them from the Kubernetes model.
- **Bring your own data platform**: set `database_offer_url`, `kafka_offer_url`, and `opensearch_offer_url` (all three together) to existing offers on the same controller. The module then skips the data platform deployment.

## Enable SSO (optional)

Set the `oauth_external_idp_integrator_config` variable with at minimum `client_id` and `client_secret` (the endpoint options default to Google). The module then deploys the integrator and relates it to DataHub. Leave it `null` to disable SSO, or relate DataHub to a Canonical Identity Platform Hydra outside the module.

## Apply and verify

Run the module with your model UUIDs and any overrides, then verify as usual:

```bash
juju run datahub-k8s/0 get-password
juju run traefik-frontend/0 show-proxied-endpoints
```

## Notes

- Generated encryption keys and any IdP credentials end up in the Terraform state. Use an encrypted or remote backend for real deployments.
- The full variable reference and test instructions live in the module READMEs in the repository.
