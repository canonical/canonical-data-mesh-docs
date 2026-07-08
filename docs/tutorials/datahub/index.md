# Get started with Charmed DataHub

This tutorial describes how to get started with Charmed DataHub. It takes you from setting up a local Juju environment, through deploying DataHub and its supporting charms, to ingesting metadata from a PostgreSQL database and exploring it in the DataHub UI.

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

1. [Set up your environment](02-environment-setup.md)
2. [Deploy supporting charms](03-deploy-supporting-charms.md)
3. [Deploy Charmed DataHub](04-deploy-datahub.md)
4. [Ingest metadata from a database](05-ingest-metadata.md)
5. [Clean up](06-cleanup.md)

```{toctree}
:hidden:
:maxdepth: 1

02-environment-setup
03-deploy-supporting-charms
04-deploy-datahub
05-ingest-metadata
06-cleanup
```
