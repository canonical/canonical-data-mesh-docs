(tutorial-airbyte-cleanup)=

# Clean up

When you are done experimenting, remove the tutorial deployment to free up resources.

## Destroy the model

Destroy the model, including its storage. This removes Airbyte and all of the supporting charms in one step:

```bash
juju destroy-model airbyte-model --destroy-storage --no-prompt
```

## Destroy the controller

If you no longer need the Juju controller:

```bash
juju destroy-controller airbyte-controller --destroy-all-models --destroy-storage --no-prompt
```

## Remove the snaps

To remove the tools installed for this tutorial:

```bash
sudo snap remove juju microk8s
```

## Next steps

- Work through the {ref}`how-to guides <how-to-airbyte-index>` for securing and observing your deployment.
- Read the {ref}`Airbyte architecture <explanation-airbyte-architecture>` explanation to understand the components behind what you just deployed.
- Check the {ref}`Airbyte reference <reference-airbyte-index>` for system requirements and Charmhub-generated configuration details.
