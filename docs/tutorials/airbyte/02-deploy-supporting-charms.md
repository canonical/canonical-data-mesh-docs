(tutorial-airbyte-deploy-supporting-charms)=

# Deploy supporting charms

In this step you deploy the supporting charms that Airbyte requires for metadata storage, workflow orchestration, and object storage.

| Requirement | Charm | Purpose |
|---|---|---|
| Database | [postgresql-k8s](https://charmhub.io/postgresql-k8s) | Stores metadata, job configurations, and sync history |
| Workflow engine | [temporal-k8s](https://charmhub.io/temporal-k8s) | Manages task queues and workflow execution |
| Admin UI | [temporal-admin-k8s](https://charmhub.io/temporal-admin-k8s) | Manages Temporal namespaces and admin tasks |
| Object storage | [minio](https://charmhub.io/minio) or [s3-integrator](https://charmhub.io/s3-integrator) | Stores sync logs, state, and artifacts |

```{note}
Use either MinIO or the S3 Integrator, not both. This tutorial uses MinIO.
```

## Deploy PostgreSQL

```bash
juju deploy postgresql-k8s --channel 14/stable --trust
juju status --watch 2s
```

```{note}
Deployment may take around 10 minutes. Expect `active` status for all units once complete.
```

## Deploy MinIO

```bash
juju deploy minio --channel ckf-1.10/stable
juju status --watch 2s
```

Deployment completes when all units are `active`.

## Deploy Temporal

```bash
juju deploy temporal-k8s --config num-history-shards=4
juju deploy temporal-admin-k8s
juju status --watch 2s
```

```{note}
Temporal requires `num-history-shards` to be a power of 2. Set it to 1024 or 2048 for a production deployment.
```

Ignore any temporary `blocked` messages; they are resolved once relations are added in the next step.

Once the supporting charms are deployed, you are ready to {ref}`deploy Charmed Airbyte <tutorial-airbyte-deploy-airbyte>`.
