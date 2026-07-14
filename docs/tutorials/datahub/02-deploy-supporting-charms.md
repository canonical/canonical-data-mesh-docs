(tutorial-datahub-deploy-supporting-charms)=

# Deploy supporting charms

DataHub relies on three backing services, each provided by an existing charm:

- [PostgreSQL](https://charmhub.io/postgresql) stores the metadata itself.
- [Kafka](https://charmhub.io/kafka) carries metadata change events and audit logs. It requires [ZooKeeper](https://charmhub.io/zookeeper).
- [OpenSearch](https://charmhub.io/opensearch) powers search and graph indexing. It requires a TLS certificates provider.

All of them run in the machine model you created in the previous step.

## Deploy the data platform

Switch to the machine model and deploy the five applications:

```bash
juju switch lxd:datahub-vm

juju deploy postgresql --channel 14/stable
juju deploy kafka --channel 3/stable
juju deploy zookeeper --channel 3/stable
juju deploy opensearch --channel 2/stable -n 2
juju deploy self-signed-certificates --channel latest/stable
```

Integrate Kafka with ZooKeeper, and OpenSearch with the certificates provider:

```bash
juju integrate kafka zookeeper
juju integrate opensearch self-signed-certificates
```

Expose the client endpoints so applications in other models can reach them:

```bash
juju expose kafka --endpoints kafka-client
juju expose opensearch --endpoints opensearch-client
```

## Wait for the applications to settle

The data platform takes 10–15 minutes to become active. You can watch progress with `juju status --watch 1s`.

## Offer the endpoints to other models

DataHub lives in a different model, so the three client endpoints must be published as offers:

```bash
juju offer postgresql:database pg-client
juju offer kafka:kafka-client kafka-client
juju offer opensearch:opensearch-client os-client
```

Verify the offers:

```bash
juju offers
```

You should see `pg-client`, `kafka-client`, and `os-client` listed. The data platform is ready; continue to {ref}`deploy DataHub <tutorial-datahub-deploy-datahub>`.
