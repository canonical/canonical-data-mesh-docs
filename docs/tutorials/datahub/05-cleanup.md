(tutorial-datahub-cleanup)=

# Clean up

When you are done experimenting, remove the tutorial deployment to free up resources.

## Destroy the models

Destroy both models, including their storage:

```bash
juju destroy-model -c microk8s datahub-k8s --destroy-storage --no-prompt
juju destroy-model -c lxd datahub-vm --destroy-storage --no-prompt
```

## Destroy the controllers

If you no longer need the Juju controllers:

```bash
juju destroy-controller microk8s --destroy-all-models --destroy-storage --no-prompt
juju destroy-controller lxd --destroy-all-models --destroy-storage --no-prompt
```

## Remove the snaps

To remove the tools installed for this tutorial:

```bash
sudo snap remove juju microk8s lxd
```

## Next steps

- Work through the {ref}`how-to guides <how-to-datahub-index>` for ingress, SSO, Trino integration, and observability.
- Read the {ref}`explanation <explanation-datahub-architecture>` section to understand the architecture behind what you just deployed.
- Check the {ref}`reference <reference-datahub-index>` section for configurations, actions, and integration details.
