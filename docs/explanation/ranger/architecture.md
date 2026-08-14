(explanation-ranger-architecture)=

# Ranger architecture

```{note}
This page is a placeholder; the explanation is planned but not yet written.
```

This page will explain the components of a Ranger deployment and the design decisions behind the charm.

Planned content:

- The Ranger admin server, its policy database, and the usersync component.
- How policies reach the query engine: the plugin embedded in Trino and its policy cache.
- The policy model: services, security zones, policies, roles, and how they map onto Juju relations.
- Where audit records are written and how they are retained.

In the meantime, see the [ranger-k8s Charmhub page](https://charmhub.io/ranger-k8s).
