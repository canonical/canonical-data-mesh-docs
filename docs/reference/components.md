(reference-components)=

# Components

The Data Mesh is composed of the following charmed operators.

| Component | Role in the solution | Charmhub |
|---|---|---|
| Trino | Federated SQL query engine; the single access layer over all connected data sources. | [trino-k8s](https://charmhub.io/trino-k8s) |
| Superset | Self-service analytics: SQL exploration, charts, and dashboards. | [superset-k8s](https://charmhub.io/superset-k8s) |
| Ranger | Fine-grained access policies and audit for the query layer. | [ranger-k8s](https://charmhub.io/ranger-k8s) |
| DataHub | Metadata catalog: discovery, ownership, and lineage. See the {ref}`DataHub reference <reference-datahub-index>`. | [datahub-k8s](https://charmhub.io/datahub-k8s) |
| Airbyte | Connector-based data ingestion from SaaS applications, databases, files, and APIs. | [airbyte-k8s](https://charmhub.io/airbyte-k8s) |
| Temporal | Workflow orchestration for ingestion and governance tasks. | [temporal-k8s](https://charmhub.io/temporal-k8s) |
| PostgreSQL | Relational storage for platform metadata and ingested data. | [postgresql-k8s](https://charmhub.io/postgresql-k8s) |
| MinIO | S3-compatible object storage for logs, state, and large datasets. | [minio](https://charmhub.io/minio) |
| Hive Metastore | Table metadata service for object-storage-backed Trino catalogs. | [hive-metastore-k8s](https://charmhub.io/hive-metastore-k8s) |
| Traefik | Kubernetes ingress for the user-facing services. | [traefik-k8s](https://charmhub.io/traefik-k8s) |

Supporting services used by individual components (for example Kafka, ZooKeeper, and OpenSearch for DataHub) are covered in the component's own section of this documentation or in the supporting charm's Charmhub page.
