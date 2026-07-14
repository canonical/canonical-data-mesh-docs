(how-to-connect-a-data-source)=

# Connect a data source to Trino

```{note}
This page is a placeholder; the guide is planned but not yet written.
```

This guide will describe how to bring an existing database into the Canonical Data Mesh as a governed, queryable catalog: from a source PostgreSQL database, through a Trino catalog, to a Ranger security zone and an automatically created Superset database connection.

Planned outline:

- Relate a PostgreSQL database to Trino to create a catalog dynamically.
- Register the catalog in a Ranger security zone (`policy` relation before `trino-catalog`).
- Verify the catalog appears as a Superset database connection.
- Verify the catalog's metadata is ingested into DataHub.
