(how-to-datahub-integrate-with-trino)=

# Integrate with Trino

This guide describes how to connect DataHub to [Trino](https://charmhub.io/trino-k8s), a distributed SQL query engine. The integration lets DataHub discover Trino catalogs and set up a scheduled metadata ingestion for each one with no manual ingestion configuration required.

## Establish the relation

Deploy Trino if you don't have it yet, then relate the two charms over the `trino-catalog` interface:

```bash
juju deploy trino-k8s --channel latest/edge
juju integrate datahub-k8s trino-k8s
```

If Trino runs in a different model, use a cross-model relation. From the Trino model:

```bash
juju offer trino-k8s:trino-catalog
```

Then, from the DataHub model:

```bash
juju consume <CONTROLLER>:admin/<TRINO_MODEL>.trino-catalog
juju integrate datahub-k8s trino-catalog
```

## Verify

In the DataHub UI, select **Data Sources**, under **Admin**, in the left navigation sidebar. You should see one source per Trino catalog, named `[juju] <catalog>-ingestion`. Each runs on a randomly assigned daily schedule between 22:00 and 06:00 UTC. After the first run completes, the catalog's schemas and tables appear in DataHub search.

## Configure ingestion filter patterns

The `trino-patterns` configuration option sets the default allow and deny regex patterns applied to schemas, tables, and views of newly created ingestion sources:

```bash
juju config datahub-k8s trino-patterns='{"schema-pattern":{"allow":[".*"],"deny":[]},"table-pattern":{"allow":[".*"],"deny":[]},"view-pattern":{"allow":[".*"],"deny":[]}}'
```

Patterns are applied only when an ingestion source is first created, so changing this option affects future catalogs, not existing sources. Existing sources can be customized freely in the DataHub UI.

## What the charm manages

The relation sets ingestion sources up; it does not own them afterwards. On every reconciliation, the charm refreshes only the details needed to reach Trino:

- The Trino host and port.
- The Trino username, and the reference to the password. The credentials themselves are held in DataHub secrets, which the charm keeps current, so a credential rotation never changes the recipe.
- The HTTP and HTTPS proxy variables derived from the model configuration. These live in the ingestion source's execution environment rather than in the recipe, so keeping them current does not interfere with your recipe edits.

Everything else is left exactly as you set it: filter patterns, the ingestion schedule, the executor, the sink, profiling, and any other setting you add to the recipe in the DataHub UI. A customized ingestion source keeps its configuration across relation changes and charm upgrades.

## Removal behavior

The charm never deletes ingestion sources or secrets:

- When a catalog disappears from the Trino relation, its ingestion source stays in place and goes stale. Delete it in the DataHub UI if you no longer want it.
- When the relation is removed entirely, every ingestion source and secret is left untouched. Re-establishing the relation reuses them as they are, instead of recreating them with default settings.
- Already ingested metadata is *not* removed; datasets stay in the catalog until they age out or are hard-deleted.

This is deliberate. Removing and re-adding the relation would otherwise destroy every customization made to the managed ingestion sources, and they would come back with default configuration only.

User-created ingestion sources and secrets are never touched. See {ref}`Integrations <reference-datahub-integrations>` for the naming conventions that identify charm-managed resources.
