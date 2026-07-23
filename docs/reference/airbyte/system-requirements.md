(reference-airbyte-system-requirements)=

# System requirements

## Software

| Requirement | Version |
|---|---|
| Juju | 3.1 or later |
| Kubernetes | A cluster supported by Juju, with a default storage class |

## Supporting charms

Airbyte depends on the following charms for metadata storage, workflow orchestration, and object storage:

| Requirement | Charm | Purpose |
|---|---|---|
| Database | [postgresql-k8s](https://charmhub.io/postgresql-k8s) | Stores metadata, job configurations, and sync history |
| Workflow engine | [temporal-k8s](https://charmhub.io/temporal-k8s) | Manages task queues and workflow execution |
| Admin UI | [temporal-admin-k8s](https://charmhub.io/temporal-admin-k8s) | Manages Temporal namespaces and admin tasks |
| Object storage | [minio](https://charmhub.io/minio) or [s3-integrator](https://charmhub.io/s3-integrator) | Stores sync logs, state, and artifacts |

## Resource sizing

For a single-host evaluation deployment (the {ref}`tutorial <tutorial-airbyte-index>` setup):

| Resource | Minimum |
|---|---|
| CPU | 2 cores |
| RAM | 8 GB |
| Disk | 20 GB |

Production sizing depends on the number and frequency of connections and the volume of data synchronized.

## Supported dependency channels

The deployment flows in this documentation are tested with:

| Charm | Channel |
|---|---|
| [postgresql-k8s](https://charmhub.io/postgresql-k8s) | 14/stable |
| [minio](https://charmhub.io/minio) | ckf-1.10/stable |
| [temporal-k8s](https://charmhub.io/temporal-k8s) | latest/edge |
| [temporal-admin-k8s](https://charmhub.io/temporal-admin-k8s) | latest/edge |
| [airbyte-k8s](https://charmhub.io/airbyte-k8s) | latest/edge |
