(tutorial-airbyte-environment-setup)=

# Set up your environment

Charmed Airbyte runs on Kubernetes and is operated with Juju. In this step you install MicroK8s and Juju, then bootstrap the controller and model used throughout the tutorial.

## Set up MicroK8s

Charmed Airbyte relies on Kubernetes as a container orchestration system. For this tutorial, you use [MicroK8s](https://microk8s.io/docs), a lightweight distribution of Kubernetes.

Install MicroK8s and give your user the required permissions by adding it to the `snap_microk8s` group and taking ownership of the `~/.kube` directory:

```bash
sudo snap install microk8s --channel 1.34-strict/stable
sudo usermod -a -G snap_microk8s $USER
mkdir -p ~/.kube
sudo chown -f -R $USER ~/.kube
newgrp snap_microk8s
```

```{note}
`newgrp` applies the new group to the current shell only. If you open a new terminal, log out and back in first so it picks up the `snap_microk8s` group.
```

Wait for MicroK8s to be ready, then enable the necessary MicroK8s add-ons:

```bash
sudo microk8s status --wait-ready
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

Since the Juju package is strictly confined, manually create the path it expects:

```bash
mkdir -p ~/.local/share
```

Juju recognises a MicroK8s cloud automatically, as you can see by running `juju clouds`:

```{terminal}
:output-only:
Cloud      Regions  Default    Type  Credentials  Source    Description
localhost  1        localhost  lxd   0            built-in  LXD Container Hypervisor
microk8s   1        localhost  k8s   1            built-in  A Kubernetes Cluster
```

If for any reason MicroK8s is not recognised, register it manually with `juju add-k8s microk8s`.

Next, make sure MicroK8s is ready after enabling the add-ons, then bootstrap a Juju controller into your MicroK8s cloud. In this tutorial, the controller is named `airbyte-controller`:

```bash
sudo microk8s status --wait-ready
juju bootstrap microk8s airbyte-controller
```

If `juju bootstrap` fails with `dial tcp 127.0.0.1:16443: connect: connection refused`, the Kubernetes API server is not ready yet. Run `sudo microk8s status --wait-ready` again and retry.

Finally, create a model on this controller. In this tutorial, the model is named `airbyte-model`; Juju creates a matching Kubernetes namespace:

```bash
juju add-model airbyte-model
```

After this, `juju status` shows something similar to:

```{terminal}
:output-only:
Model          Controller          Cloud/Region        Version  SLA          Timestamp
airbyte-model  airbyte-controller  microk8s/localhost  3.6.12   unsupported  12:45:50+03:00

Model "admin/airbyte-model" is empty.
```

You are now ready to {ref}`deploy the supporting charms <tutorial-airbyte-deploy-supporting-charms>`.
