(tutorial-datahub-deploy-datahub)=

# Deploy DataHub

With the data platform running, you can now deploy DataHub itself in the Kubernetes model and connect it to PostgreSQL, Kafka, and OpenSearch.

## Consume the data platform offers

Switch to the Kubernetes model and consume the offers you created in the previous step:

```bash
juju switch microk8s:datahub-k8s

juju consume lxd:admin/datahub-vm.pg-client pg-client
juju consume lxd:admin/datahub-vm.kafka-client kafka-client
juju consume lxd:admin/datahub-vm.os-client os-client
```

## Create the encryption keys secret

DataHub needs two encryption keys, one for the metadata service (GMS) and one for the frontend. Generate random keys and store them in a Juju user secret:

```bash
GMS_KEY=$(openssl rand -base64 32)
FRONTEND_KEY=$(openssl rand -base64 32)
juju add-secret datahub-encryption-keys gms-key=$GMS_KEY frontend-key=$FRONTEND_KEY
```

The command prints the secret ID (a URI starting with `secret:`). Note it down; the next step passes it to the charm.

## Deploy DataHub

Deploy the charm, pointing it at the secret, then grant the charm access to the secret:

```bash
juju deploy datahub-k8s --channel latest/edge --config encryption-keys-secret-id=<SECRET_ID>
juju grant-secret datahub-encryption-keys datahub-k8s
```

Integrate DataHub with the three data platform endpoints:

```bash
juju integrate datahub-k8s pg-client
juju integrate datahub-k8s kafka-client
juju integrate datahub-k8s os-client
```

## Wait for DataHub to initialize

On first startup, the charm pulls three container images (actions, frontend, and GMS, around 1 GB combined), then DataHub prepares its database schema, creates its OpenSearch indices and Kafka topics, and runs a one-time system upgrade job before the services start. The image pulls are usually the bulk of the wait and depend heavily on network speed; the DataHub-specific setup that follows is comparatively fast. Expect up to 20 minutes on a normal connection, more on a slow one. You can watch progress with `juju status --watch 1s`.

## Log in to DataHub

Retrieve the auto-generated admin password:

```bash
juju run datahub-k8s/0 get-password
```

The default admin username is `datahub`.

Find the unit's IP address in the output of `juju status` (the `Address` column of the `datahub-k8s/0` unit), then open the DataHub UI in your browser:

```text
http://<UNIT_ADDRESS>:9002
```

Log in with the username `datahub` and the password from the action. You should see the DataHub home page with an empty catalog.

```{note}
Accessing the unit IP directly is fine for a local tutorial. For anything
beyond that, expose DataHub through an ingress - see
{ref}`Expose DataHub with ingress <how-to-datahub-expose-with-ingress>`.
```

DataHub is up and running. Continue to {ref}`ingest metadata from a database <tutorial-datahub-ingest-metadata>`.
