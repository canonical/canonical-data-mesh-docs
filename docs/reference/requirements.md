(reference-requirements)=

# Requirements

```{note}
This page is a placeholder; detailed sizing guidance is planned.
```

Platform requirements for the Canonical Data Mesh:

| Requirement | Version |
|---|---|
| Kubernetes | 1.25 or later |
| Juju | 3/stable |

Some components have additional requirements:

- OpenSearch (a DataHub dependency) requires a machine cloud and specific kernel parameters on its host, see the {ref}`DataHub system requirements <reference-datahub-system-requirements>`.
- Cross-model relations are used throughout; a single controller with both a Kubernetes and a machine cloud registered is the recommended topology.

Planned content:

- Per-component resource sizing for evaluation and production deployments.
- Supported charm channels and version compatibility.
