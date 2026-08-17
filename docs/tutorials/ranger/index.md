(tutorial-ranger-index)=

# Get started with Ranger

```{note}
This page is a placeholder; the tutorial is planned but not yet written.
```

[Apache Ranger](https://ranger.apache.org/) provides centralized access control and audit for the query layer. This tutorial will describe how to get started with Charmed Ranger, the Kubernetes operator that deploys and manages it.

Planned outline:

1. Set up the environment (MicroK8s and Juju).
2. Deploy the Ranger admin server and its database.
3. Integrate Ranger with Trino to install the policy plugin.
4. Create users, groups, and a first access policy.
5. See the policy take effect on a query, and read the audit trail.
6. Clean up.

In the meantime, see the [ranger-k8s Charmhub page](https://charmhub.io/ranger-k8s).
