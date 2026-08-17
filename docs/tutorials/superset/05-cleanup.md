(tutorial-superset-cleanup)=

# Clean up

When you are done experimenting, remove the tutorial deployment to free up resources.

## If you used the Multipass VM

If you followed the {ref}`prerequisites <tutorial-superset-index>` and ran this tutorial in the `superset-tutorial` Multipass VM, deleting the VM removes everything at once, including Juju, MicroK8s, and all deployed charms:

```bash
multipass delete superset-tutorial --purge
```

This is the fastest way to clean up and needs no further steps below.

## If you ran this on your own machine

If you ran the tutorial directly on a host rather than in a disposable VM, remove each piece individually instead.

### Destroy the controller

Destroying the controller removes its models, including all the data stored in PostgreSQL and Redis:

```bash
juju destroy-controller superset-controller --destroy-all-models --destroy-storage --no-prompt
```

### Remove the snaps

To remove the tools installed for this tutorial:

```bash
sudo snap remove juju microk8s
```

## Next steps

- Work through the {ref}`how-to guides <how-to-superset-index>` for ingress, single sign-on, Trino integration, alerts and reports, and observability.
- Read the {ref}`explanation <explanation-superset-architecture>` section to understand the architecture behind what you just deployed.
- Check the {ref}`reference <reference-superset-index>` section for supported feature flags, integrations, and system requirements.
