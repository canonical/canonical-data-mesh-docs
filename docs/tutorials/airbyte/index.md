(tutorial-airbyte-index)=

# Get started with Airbyte

[Airbyte](https://airbyte.io/) is an open-source data integration platform that extracts and loads data from a wide range of sources into data warehouses, data lakes, and data meshes. This tutorial describes how to get started with Charmed Airbyte, the Kubernetes operator that deploys and manages it. It takes you from setting up a local Juju environment, through deploying the supporting charms Airbyte depends on, to deploying Airbyte itself and reaching a fully operational instance.

Airbyte relies on three backing services: PostgreSQL for metadata storage, Temporal for workflow orchestration, and object storage (MinIO or an S3-compatible backend) for logs, state, and artifacts. This tutorial deploys all of them alongside Airbyte on a single MicroK8s cloud.

By the end of this tutorial, you will have a running Charmed Airbyte deployment integrated with its supporting charms and ready to configure connections.

## Prerequisites

Before you begin, make sure the machine has:

- Ubuntu 22.04 LTS or later
- [Snap](https://snapcraft.io/docs/installing-snapd) installed
- At least 8 GB of RAM
- 2 CPU cores
- At least 20 GB of available disk space

## Tutorial steps

1. {ref}`Set up your environment <tutorial-airbyte-environment-setup>`: install MicroK8s and Juju, and bootstrap the controller and model this tutorial uses.
2. {ref}`Deploy supporting charms <tutorial-airbyte-deploy-supporting-charms>`: deploy PostgreSQL, MinIO, and Temporal.
3. {ref}`Deploy Charmed Airbyte <tutorial-airbyte-deploy-airbyte>`: deploy Airbyte and integrate it with its supporting charms.
4. {ref}`Run your first sync <tutorial-airbyte-run-a-sync>`: open the UI and move data with Airbyte's built-in test connectors.
5. {ref}`Clean up <tutorial-airbyte-cleanup>`

```{toctree}
:hidden:
:maxdepth: 1

01-environment-setup
02-deploy-supporting-charms
03-deploy-airbyte
04-run-a-sync
05-cleanup
```
