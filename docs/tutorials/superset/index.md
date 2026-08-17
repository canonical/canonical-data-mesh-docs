(tutorial-superset-index)=

# Get started with Superset

[Apache Superset](https://superset.apache.org/) is an open source data exploration and visualization platform. This tutorial describes how to get started with Charmed Superset, the Kubernetes operator that deploys and manages it. It takes you from setting up a local Juju environment, through deploying Superset and its supporting charms, to building your first dashboard in the Superset UI.

Superset relies on two backing services: PostgreSQL, which stores its metadata (users, dashboard definitions, saved queries, and the action log), and Redis, which acts as the cache and the message broker between the web server and its asynchronous workers. Both run on the same Kubernetes cloud as Superset itself, so this tutorial only needs a single MicroK8s cloud.

By the end of this tutorial, you will have a working Superset instance and a published dashboard built from example data.

## Prerequisites

Before you begin, make sure your machine has:

- Ubuntu 22.04 LTS or later
- [Snap](https://snapcraft.io/docs/installing-snapd) installed
- 2 CPU threads, 4 recommended
- At least 8 GB of RAM
- At least 40 GB of available disk space

```{note}
This tutorial installs MicroK8s and Juju on the host. To keep your machine
clean, run it in a disposable VM instead. [Multipass](https://multipass.run/)
is a quick way to get one:

    sudo snap install multipass
    multipass launch --name superset-tutorial --cpus 4 --memory 8G --disk 40G noble
    multipass shell superset-tutorial
```

## Tutorial steps

1. {ref}`Set up your environment <tutorial-superset-environment-setup>`: install MicroK8s and bootstrap a Juju controller and model.
2. {ref}`Deploy supporting charms <tutorial-superset-deploy-supporting-charms>`: deploy the PostgreSQL and Redis charms that Superset requires.
3. {ref}`Deploy Superset <tutorial-superset-deploy-superset>`: deploy the charm, integrate it with its backing services, and log in to the UI.
4. {ref}`Create a dashboard <tutorial-superset-create-a-dashboard>`: load example data and publish your first dashboard.
5. {ref}`Clean up <tutorial-superset-cleanup>`

```{toctree}
:hidden:
:maxdepth: 1

01-environment-setup
02-deploy-supporting-charms
03-deploy-superset
04-create-a-dashboard
05-cleanup
```
