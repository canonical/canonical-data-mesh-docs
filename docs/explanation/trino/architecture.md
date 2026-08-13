(explanation-trino-architecture)=

# Trino architecture

```{note}
This page is a placeholder; the explanation is planned but not yet written.
```

This page will explain the components of a Trino deployment and the design decisions behind the charm.

Planned content:

- Coordinator and worker roles, and how the charm deploys them as separate applications.
- Catalogs: how connectors are configured and how catalogs are shared with other charms over the `trino-catalog` relation.
- Query authorization: how the Ranger plugin is installed and how user impersonation works.
- Query load isolation: resource groups and session property managers.

In the meantime, see the [trino-k8s Charmhub page](https://charmhub.io/trino-k8s).
