(tutorial-datahub-index)=

# Get started with DataHub

This tutorial describes how to get started with DataHub. It takes you from setting up a local Juju environment, through deploying DataHub and its supporting charms, to ingesting metadata from a PostgreSQL database and exploring it in the DataHub UI.

DataHub relies on three backing services: PostgreSQL for metadata storage, Kafka for event streaming, and OpenSearch for search and graph indexing. OpenSearch runs on machines rather than Kubernetes, so this tutorial builds a small two-cloud environment on a single host: an LXD cloud for the data platform and a MicroK8s cloud for DataHub itself. This mirrors how production deployments are structured, just at a smaller scale.

By the end of this tutorial, you will have a working DataHub instance and a searchable data catalog containing the schema of a real database table.

## Prerequisites

Before you begin, make sure you have:

- Ubuntu 22.04 LTS or later, on a physical machine or a VM
- [Snap](https://snapcraft.io/docs/installing-snapd) installed
- 8 CPU threads
- At least 24 GB of RAM
- At least 80 GB of available disk space

The DataHub stack is memory-hungry: it runs three JVM-based workloads alongside Kafka and OpenSearch. Machines below these requirements may see workloads restart or fail to settle.

## Tutorial steps

1. {ref}`Set up your environment <tutorial-datahub-environment-setup>`: install LXD and MicroK8s, and bootstrap the two Juju controllers this tutorial uses.
2. {ref}`Deploy supporting charms <tutorial-datahub-deploy-supporting-charms>`: deploy PostgreSQL, Kafka, and OpenSearch, and offer them for cross-model access.
3. {ref}`Deploy DataHub <tutorial-datahub-deploy-datahub>`: deploy DataHub, relate it to its backing services, and log in to the UI.
4. {ref}`Ingest metadata from a database <tutorial-datahub-ingest-metadata>`: create a demo table and pull its schema into the DataHub catalog.
5. {ref}`Clean up <tutorial-datahub-cleanup>`

```{toctree}
:hidden:
:maxdepth: 1

01-environment-setup
02-deploy-supporting-charms
03-deploy-datahub
04-ingest-metadata
05-cleanup
```
