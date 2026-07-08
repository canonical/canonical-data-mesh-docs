# Canonical Data Mesh

The Canonical Data Mesh is a composable data platform solution built on trusted open source projects. It deploys a governed, self-service data platform where teams can ingest, discover, query, and visualize data under centralized governance and access control.

The solution combines charmed operators for Apache Superset, Trino, Apache Ranger, DataHub, Airbyte, Temporal, PostgreSQL, Hive Metastore, and their dependencies, wired together through Juju relations. Data is ingested from source systems, queried through a federated layer, and explored through dashboards and a dedicated catalog, with every action subject to fine-grained access control.

The Canonical Data Mesh is for platform and data teams that want self-service analytics and cross-system reporting without building and operating a bespoke data stack.

## In this documentation

`````{grid} 1 1 2 2

````{grid-item-card} Tutorials
:link: tutorials/index
:link-type: doc

**Get started**: deploy the Data Mesh or an individual component for the first time and see a real result.
````

````{grid-item-card} How-to guides
:link: how-to/index
:link-type: doc

**Step-by-step guides**: deploy, integrate, govern, and operate the platform and its components.
````

`````

`````{grid} 1 1 2 2

````{grid-item-card} Reference
:link: reference/index
:link-type: doc

**Technical information**: architecture, components, requirements, and per-charm reference material.
````

````{grid-item-card} Explanation
:link: explanation/index
:link-type: doc

**Concepts**: the ideas behind the data mesh, its governance model, and the design of its components.
````

`````

## How this documentation is organized

This site is the single home for the Canonical Data Mesh documentation with each section of the four above containing solution-level pages first, followed by a subsection for each charm.

Charm configuration options, actions, and integration endpoints are enumerated on each charm's Charmhub page (generated from the charm itself) and are not duplicated here.

## Project and community

The Canonical Data Mesh is a member of the Ubuntu family. It's an open source project that warmly welcomes community contributions, suggestions, fixes, and constructive feedback.

- [Code of conduct](https://ubuntu.com/community/ethos/code-of-conduct)
- [Join the Discourse forum](https://discourse.charmhub.io/)
- [Contribute and report bugs](https://github.com/canonical/canonical-data-mesh-docs)

```{toctree}
:hidden:
:maxdepth: 1

tutorials/index
how-to/index
reference/index
explanation/index
```

```{toctree}
:hidden:
:maxdepth: 1

release-notes/index
contribute/index
```
