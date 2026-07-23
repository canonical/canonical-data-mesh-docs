(tutorial-airbyte-environment-setup)=

# Set up your environment

Charmed Airbyte runs on Kubernetes and is operated with Juju. In this step you install MicroK8s and Juju, then bootstrap the controller and model used throughout the tutorial.

## Set up MicroK8s

Charmed Airbyte relies on Kubernetes as a container orchestration system. For this tutorial, you use [MicroK8s](https://microk8s.io/docs), a lightweight distribution of Kubernetes.

Install MicroK8s and give your user the required permissions by adding it to the `snap_microk8s` group and taking ownership of the `~/.kube` directory:

```bash
sudo snap install microk8s --channel 1.34-strict/stable
newgrp snap_microk8s
sudo usermod -a -G snap_microk8s $USER
sudo chown -f -R $USER ~/.kube
```

Enable the necessary MicroK8s add-ons:

```bash
sudo microk8s enable hostpath-storage dns
```

For convenience, set up a short alias for the Kubernetes CLI:

```bash
sudo snap alias microk8s.kubectl kubectl
```

## Set up Juju

Charmed Airbyte uses Juju as the orchestration engine for software operators. Install it and connect it to your MicroK8s cloud.

Install `juju` from a snap:

```bash
sudo snap install juju --channel 3.6/stable
```

```{note}
This charm requires Juju 3.1 or later.
```

Since the Juju package is strictly confined, manually create the path it expects:

```bash
mkdir -p ~/.local/share
```

Juju recognises a MicroK8s cloud automatically, as you can see by running `juju clouds`:

```text
Cloud      Regions  Default    Type  Credentials  Source    Description
localhost  1        localhost  lxd   0            built-in  LXD Container Hypervisor
microk8s   1        localhost  k8s   1            built-in  A Kubernetes Cluster
```

If for any reason MicroK8s is not recognised, register it manually with `juju add-k8s microk8s`.

Next, bootstrap a Juju controller into your MicroK8s cloud. In this tutorial, the controller is named `airbyte-controller`:

```bash
juju bootstrap microk8s airbyte-controller
```

Finally, create a model on this controller. In this tutorial, the model is named `airbyte-model`; Juju creates a matching Kubernetes namespace:

```bash
juju add-model airbyte-model
```

After this, `juju status` shows something similar to:

```text
Model          Controller          Cloud/Region        Version  SLA          Timestamp
airbyte-model  airbyte-controller  microk8s/localhost  3.6.12   unsupported  12:45:50+03:00

Model "admin/airbyte-model" is empty.
```

You are now ready to {ref}`deploy the supporting charms <tutorial-airbyte-deploy-supporting-charms>`.
