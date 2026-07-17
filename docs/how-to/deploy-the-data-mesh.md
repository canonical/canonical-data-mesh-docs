(how-to-deploy-the-data-mesh)=

# Deploy the Canonical Data Mesh

```{note}
This page is a placeholder; the guide is planned but not yet written.
```

This guide will describe how to deploy the Canonical Data Mesh for production-like environments using the Terraform product modules of the component charms.

Planned outline:

- Prerequisites: controller and cloud topology (Kubernetes and machine clouds, cross-model offers).
- Deploy the query and governance core: PostgreSQL, Trino, Ranger, and Superset.
- Deploy the metadata layer: DataHub and its data platform (PostgreSQL, Kafka, OpenSearch).
- Deploy the ingestion layer: Airbyte, MinIO, and Temporal.
- Expose the user-facing services with ingress and TLS.
- Enable single sign-on across the platform.
