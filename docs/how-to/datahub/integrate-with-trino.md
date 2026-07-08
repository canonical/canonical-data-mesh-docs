# Integrate with Trino

This guide describes how to connect DataHub to [Charmed Trino](https://charmhub.io/trino-k8s), a distributed SQL query engine. The integration lets DataHub discover Trino catalogs and set up a scheduled metadata ingestion for each one with no manual ingestion configuration required.

## Establish the relation

Deploy Trino if you don't have it yet, then relate the two charms over the `trino-catalog` interface:

```bash
juju deploy trino-k8s --channel latest/edge
juju integrate datahub-k8s:trino-catalog trino-k8s
```

If Trino runs in a different model, use a cross-model relation. From the Trino model:

```bash
juju offer trino-k8s:trino-catalog
```

Then, from the DataHub model:

```bash
juju consume <CONTROLLER>:admin/<TRINO_MODEL>.trino-catalog
juju integrate datahub-k8s:trino-catalog trino-catalog
```

## Verify

In the DataHub UI, select **Ingestion**. You should see one ingestion source per Trino catalog, named `[juju] <catalog>-ingestion`. Each runs on a randomly assigned daily schedule between 22:00 and 06:00 UTC. After the first run completes, the catalog's schemas and tables appear in DataHub search.

## Configure ingestion filter patterns

The `trino-patterns` configuration option sets the default allow and deny regex patterns applied to schemas, tables, and views of newly created ingestion sources:

```bash
juju config datahub-k8s trino-patterns='{"schema-pattern":{"allow":[".*"],"deny":[]},"table-pattern":{"allow":[".*"],"deny":[]},"view-pattern":{"allow":[".*"],"deny":[]}}'
```

Patterns are applied only when an ingestion source is first created, so changing this option affects future catalogs, not existing sources. Existing sources can be customized freely in the DataHub UI.

## What the charm manages

On every reconciliation, the charm overwrites the following fields of each managed ingestion source:

- Access tokens and Trino credentials (stored as DataHub secrets)
- Trino host, port, and catalog name
- HTTP and HTTPS proxy variables derived from the model configuration

The following are set once at creation and then left alone, so your UI customizations survive:

- Filter patterns (from `trino-patterns`)
- The ingestion schedule

The schedule, description, executor, and any additional arguments can be edited in the DataHub UI without interference from the charm.

## Removal behavior

- When a catalog disappears from the Trino relation, its ingestion source is deleted automatically.
- When the relation is removed entirely, all Juju-managed ingestion sources and their secrets are cleaned up.
- Already ingested metadata is *not* removed; datasets stay in the catalog until they age out or are hard-deleted.

User-created ingestion sources and secrets are never touched. See [Integrations](../../reference/datahub/integrations.md) for the naming conventions that identify charm-managed resources.
