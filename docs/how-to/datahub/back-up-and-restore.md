# Back up and restore

DataHub itself is stateless: everything it serves is derived from its backing services. Backing up a DataHub deployment therefore means backing up two things:

- **Metadata**, stored in PostgreSQL. This is the source of truth.
- **Search and graph indices**, stored in OpenSearch. These are derived from the metadata and can be rebuilt from it.

## Back up the metadata

Use the PostgreSQL charm's built-in backup actions, which store backups in S3-compatible storage. Follow the [Charmed PostgreSQL backup guide](https://canonical-charmed-postgresql.readthedocs-hosted.com/14/how-to/back-up-and-restore/).

## Back up the indices

OpenSearch indices can be backed up with the OpenSearch charm's snapshot actions, as described in the [Charmed OpenSearch backup guide](https://charmhub.io/opensearch/docs/h-create-backup).

However, since the indices are derived data, it is often easier to skip index backups entirely and rebuild them after a restore.

## Rebuild the indices

The charm exposes DataHub's [index restoration](https://docs.datahub.com/docs/how/restore-indices/) as the `reindex` action. It fetches the latest version of each metadata aspect from PostgreSQL and rebuilds the search and graph indices:

```bash
juju run datahub-k8s/leader reindex
```

To delete the existing indices before rebuilding, use the `clean` parameter:

```bash
juju run datahub-k8s/leader reindex clean=true
```

Run `reindex` whenever the indices may be out of sync with the metadata store - after restoring a PostgreSQL backup, after migrating OpenSearch, or when search results are missing or stale.

## Restore procedure

1. Restore the PostgreSQL database from backup using the PostgreSQL charm's restore action.
2. Either restore the OpenSearch snapshot, or rebuild the indices:

```bash
juju run datahub-k8s/leader reindex clean=true
```

3. Verify by searching for a known dataset in the DataHub UI.
