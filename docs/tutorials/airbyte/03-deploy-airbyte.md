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

## Integrate Airbyte with MinIO

Airbyte requires object storage for logs, artifacts, and state. Add the relation between MinIO and Airbyte:

```bash
juju relate minio airbyte-k8s
```

Expected status after the relation settles:

```text
airbyte-k8s/0: active | waiting for database connection
```

## Integrate Airbyte with PostgreSQL

Airbyte depends on PostgreSQL to store metadata, configuration, and job history. Add the relation between PostgreSQL and Airbyte:

```bash
juju relate postgresql-k8s airbyte-k8s
```

Airbyte transitions to a new blocked state until Temporal is related:

```text
temporal relation not ready
```

## Integrate Airbyte with Temporal

Airbyte depends on two Temporal charms:

- `temporal-k8s` — the Temporal workflow engine
- `temporal-admin-k8s` — provides UI and admin capabilities

Add the relations:

```bash
juju relate temporal-k8s:db postgresql-k8s:database
juju relate temporal-k8s:visibility postgresql-k8s:database
juju relate temporal-k8s:admin temporal-admin-k8s:admin
```

After all relations and configurations are applied, check the status:

```bash
juju status
```

All applications (`airbyte-k8s`, `temporal-k8s`, `temporal-admin-k8s`, `postgresql-k8s`, `minio`) should eventually show `active` status. At this point, Airbyte is fully operational:

```text
Model          Controller          Cloud/Region        Version  SLA          Timestamp
airbyte-model  airbyte-controller  microk8s/localhost  3.6.11   unsupported  14:00:29+03:00

App                   Version                Status  Scale  Charm                   Channel          Rev  Address   Exposed  Message
airbyte-k8s           v1.7.0                 active      1  airbyte-k8s             latest/edge       18  10.x.x.x  no
airbyte-webhooks-k8s                         active      1  airbyte-webhooks-charm  latest/edge       12  10.x.x.x  no
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

## Next steps

- Follow {ref}`Secure Airbyte deployments <how-to-airbyte-secure-deployments>` to add TLS and authentication to your deployment.
- Read the {ref}`Airbyte architecture <explanation-airbyte-architecture>` explanation to understand the components behind what you just deployed.
- Check the {ref}`Airbyte reference <reference-airbyte-index>` for system requirements and Charmhub-generated configuration details.
