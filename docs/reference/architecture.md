(reference-architecture)=

# Architecture

```{note}
This page is a work in progress; an architecture diagram and the integration matrix are planned.
```

The Data Mesh operates as an integrated lifecycle where data flows through distinct stages, each handled by a dedicated component:

- **Ingestion**: Airbyte connects to source systems and loads data into platform storage.
- **Storage**: PostgreSQL and MinIO provide relational and object persistence.
- **Access**: Trino supplies a unified SQL layer over all connected catalogs.
- **Governance**: Ranger enforces access policies at the query layer; DataHub catalogs metadata, ownership, and lineage.
- **Consumption**: Superset provides exploration, charts, and dashboards.
- **Orchestration**: Temporal automates ingestion and governance workflows.
- **Observability**: the Canonical Observability Stack tracks platform health.

Key integration points:

- PostgreSQL to Trino: source databases become Trino catalogs dynamically over a relation.
- Trino to Superset: catalogs appear as Superset database connections with managed credentials.
- Trino to Ranger: queries are authorized against Ranger policies and audited.
- Trino to DataHub: catalog metadata is ingested on a schedule managed by the charms.
