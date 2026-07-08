# Set up your environment

DataHub requires a multi-cloud Juju setup: the DataHub charm runs on Kubernetes, while OpenSearch requires a machine cloud. In this step you install LXD and MicroK8s on one host, bootstrap a Juju controller on each, and create the two models used throughout the tutorial.

## Install LXD

LXD provides the machine cloud that hosts PostgreSQL, Kafka, and OpenSearch.

```bash
sudo snap install lxd --channel 5.21/stable
sudo adduser $USER lxd
newgrp lxd
lxd init --auto
```

## Install MicroK8s

MicroK8s provides the Kubernetes cloud that hosts DataHub.

```bash
sudo snap install microk8s --channel 1.34-strict/stable
sudo usermod -aG snap_microk8s $USER
newgrp snap_microk8s
microk8s enable dns hostpath-storage
```

Wait until all MicroK8s services report as running:

```bash
microk8s status --wait-ready
```

## Configure kernel parameters

OpenSearch requires a set of kernel parameters on its host. Apply them once:

```bash
echo -e "vm.max_map_count=262144\nvm.swappiness=0\nnet.ipv4.tcp_retries2=5\nfs.file-max=1048576" | sudo tee /etc/sysctl.d/99-charm-dev.conf
sudo sysctl --system
```

## Install Juju and bootstrap the controllers

Install Juju and bootstrap one controller per cloud:

```bash
sudo snap install juju --channel 3/stable

juju bootstrap lxd lxd
juju bootstrap microk8s microk8s
```

Create a machine model for the data platform and a Kubernetes model for DataHub:

```bash
juju add-model -c lxd datahub-vm
juju add-model -c microk8s datahub-k8s
```

Verify that both controllers and models exist:

```bash
juju controllers
juju models -c lxd
juju models -c microk8s
```

You should see the `lxd` and `microk8s` controllers, each with its model. You are now ready to [deploy the supporting charms](03-deploy-supporting-charms.md).

```{note}
A single controller with both clouds registered also works and is what the
Terraform product module uses. Two controllers keep the tutorial steps
symmetrical and easy to follow.
```
