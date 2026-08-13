(tutorial-superset-environment-setup)=

# Set up your environment

Superset and its backing services all run on Kubernetes. In this step you install MicroK8s, bootstrap a Juju controller on it, and create the model used throughout the tutorial.

## Install MicroK8s

MicroK8s is a lightweight Kubernetes distribution that runs on a single host. Install it and grant your user access:

```bash
sudo snap install microk8s --channel 1.34-strict/stable
sudo usermod -aG snap_microk8s $USER
newgrp snap_microk8s
sudo microk8s enable dns hostpath-storage
```

Wait until all MicroK8s services report as running:

```bash
microk8s status --wait-ready
```

Optionally, alias the Kubernetes CLI so you can call it as `kubectl`:

```bash
sudo snap alias microk8s.kubectl kubectl
```

## Install Juju and bootstrap the controller

Juju is the orchestration engine that drives the charms. Install it and bootstrap a controller into your MicroK8s cloud:

```bash
sudo snap install juju --channel 3/stable
juju bootstrap microk8s superset-controller
```

Juju recognizes a MicroK8s cloud automatically. If it does not appear in `juju clouds`, register it manually with `juju add-k8s microk8s`.

## Create a model

Create the model that holds the tutorial deployment. Juju creates a Kubernetes namespace with the same name:

```bash
juju add-model superset-tutorial
```

Verify the result:

```bash
juju status
```

```text
Model              Controller           Cloud/Region        Version  SLA          Timestamp
superset-tutorial  superset-controller  microk8s/localhost  3.6.12   unsupported  16:05:03+01:00

Model "admin/superset-tutorial" is empty.
```

Your environment is ready. Continue to {ref}`deploy the supporting charms <tutorial-superset-deploy-supporting-charms>`.
