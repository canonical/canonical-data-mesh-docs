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

Deployment may take around 10 minutes. Expect `active` status for all units once complete.

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

At this stage, PostgreSQL and MinIO reach `active`, while the two Temporal charms stay `blocked` because they have no relations yet. This is expected; you add the relations in the next step. Your `juju status` looks similar to:

```{terminal}
:output-only:
App                 Version                Status   Scale  Charm               Channel          Rev  Address   Exposed  Message
minio               res:oci-image@7f2474f  active       1  minio               ckf-1.10/stable  459  10.x.x.x  no
postgresql-k8s      14.15                  active       1  postgresql-k8s      14/stable        495  10.x.x.x  no
temporal-admin-k8s                         blocked      1  temporal-admin-k8s  latest/edge       13  10.x.x.x  no       admin:temporal relation: not available
temporal-k8s                               blocked      1  temporal-k8s        latest/edge       45  10.x.x.x  no       database relation not ready

Unit                   Workload  Agent  Address   Ports          Message
minio/0*               active    idle   10.x.x.x  9000-9001/TCP
postgresql-k8s/0*      active    idle   10.x.x.x
temporal-admin-k8s/0*  blocked   idle   10.x.x.x                 admin:temporal relation: not available
temporal-k8s/0*        blocked   idle   10.x.x.x                 database relation not ready
```

Any `blocked` message other than these two is unexpected and worth investigating before continuing.

Once the supporting charms are deployed, you are ready to {ref}`deploy Charmed Airbyte <tutorial-airbyte-deploy-airbyte>`.
