(tutorial-airbyte-deploy-airbyte)=

# Deploy Charmed Airbyte

In this step you deploy the Charmed Airbyte application and integrate it with the supporting charms deployed in the previous step.

## Deploy Charmed Airbyte

Deploy Airbyte using the official charm:

```bash
juju deploy airbyte-k8s --channel edge --trust
```

Verify the deployment:

```bash
juju status --watch 2s
```

Initially, Airbyte is in a `blocked` state with a message such as:

```text
database relation not ready
```

You add the relations in the next steps.

## Integrate Airbyte with PostgreSQL

Airbyte depends on PostgreSQL to store metadata, configuration, and job history. Add the relation between PostgreSQL and Airbyte:

```bash
juju integrate postgresql-k8s airbyte-k8s
```

The database relation is satisfied, and Airbyte moves on to check its object storage:

```text
minio relation not ready
```

## Integrate Airbyte with MinIO

Airbyte requires object storage for logs, artifacts, and state. Add the relation between MinIO and Airbyte:

```bash
juju integrate minio airbyte-k8s
```

With both PostgreSQL and MinIO related, the `airbyte-bootloader` container runs its one-time initialization; you may briefly see `waiting for airbyte-auth-secrets` while it creates the auth secret, after which `airbyte-k8s/0` settles to `active`.

## Set up Temporal

Airbyte offloads workflow execution to Temporal. It reaches Temporal through the `temporal-host` configuration option (default `temporal-k8s:7233`), not through a Juju relation, so there is no direct Airbyte-to-Temporal integration to add. You only need Temporal itself to be operational, which means relating it to its own backends:

- `temporal-k8s` — the Temporal workflow engine, which stores its default and visibility data in PostgreSQL
- `temporal-admin-k8s` — registers the Temporal namespace and provides admin capabilities

Add the relations:

```bash
juju integrate temporal-k8s:db postgresql-k8s:database
juju integrate temporal-k8s:visibility postgresql-k8s:database
juju integrate temporal-k8s:admin temporal-admin-k8s:admin
```

Because Airbyte's default `temporal-host` already points at `temporal-k8s:7233`, Airbyte connects to Temporal automatically once Temporal reaches `active`.

After all relations are applied, check the status:

```bash
juju status
```

All applications (`airbyte-k8s`, `temporal-k8s`, `temporal-admin-k8s`, `postgresql-k8s`, `minio`) should eventually show `active` status. At this point, Airbyte is fully operational:

```text
Model          Controller          Cloud/Region        Version  SLA          Timestamp
airbyte-model  airbyte-controller  microk8s/localhost  3.6.11   unsupported  14:00:29+03:00

App                   Version                Status  Scale  Charm                   Channel          Rev  Address   Exposed  Message
airbyte-k8s           v1.7.0                 active      1  airbyte-k8s             latest/edge       18  10.x.x.x  no
minio                 res:oci-image@7f2474f  active      1  minio                   ckf-1.10/stable  459  10.x.x.x  no
temporal-admin-k8s    1.23.1                 active      1  temporal-admin-k8s      latest/edge       13  10.x.x.x  no
temporal-k8s          1.23.1                 active      1  temporal-k8s            latest/edge       45  10.x.x.x  no

Unit                    Workload  Agent  Address   Ports          Message
airbyte-k8s/0*          active    idle   10.x.x.x
minio/6*                active    idle   10.x.x.x  9000-9001/TCP
temporal-admin-k8s/0*   active    idle   10.x.x.x
temporal-k8s/0*         active    idle   10.x.x.x

Integration provider     Requirer                 Interface          Type     Message
airbyte-k8s:airbyte-peer  airbyte-k8s:airbyte-peer  airbyte            peer
minio:object-storage      airbyte-k8s:object-storage  object-storage   regular
postgresql:database       airbyte-k8s:db            postgresql_client  regular
postgresql:database       temporal-k8s:db           postgresql_client  regular
postgresql:database       temporal-k8s:visibility   postgresql_client  regular
temporal-admin-k8s:admin  temporal-k8s:admin        temporal           regular
temporal-admin-k8s:peer   temporal-admin-k8s:peer   temporal-admin     peer
temporal-k8s:peer         temporal-k8s:peer         temporal           peer
```

Charmed Airbyte is up and running. Continue to {ref}`run your first sync <tutorial-airbyte-run-a-sync>`.
