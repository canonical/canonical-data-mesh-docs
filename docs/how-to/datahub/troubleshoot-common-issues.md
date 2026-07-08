# Troubleshoot common issues

This guide collects the most common failure modes of a Charmed DataHub deployment and how to diagnose them.

## Check the charm and workload status first

```bash
juju status --relations
juju debug-log --include datahub-k8s
```

A blocked status message usually names the missing piece - for example a missing relation, a missing secret grant, or `OIDC requires an HTTPS ingress URL` (see [Enable single sign-on](enable-sso.md)).

## Ingestion fails or `/graphql` returns 500

If DataHub loads but features such as **Ingestion** fail, and requests to `/graphql` return a `500` error, the OpenSearch cross-model offer is typically blocked on the provider side. Check the offer status in the OpenSearch model and ensure the offer is accepted:

```bash
juju switch <MACHINE_MODEL>
juju status --relations
```

## Search results are missing or stale

If search results are missing after a migration, restore, or upgrade, the indices are out of sync with the metadata store. Rebuild them:

```bash
juju run datahub-k8s/leader reindex
```

See [Back up and restore](back-up-and-restore.md) for details.

## GMS fails to initialize

On startup, the GMS container runs several initialization scripts: PostgreSQL setup, OpenSearch index creation, and schema migration. Each script logs to `/tmp` inside the `datahub-gms` container.

List the available logs:

```bash
kubectl -n <NAMESPACE> exec -c datahub-gms datahub-k8s-0 -- ls /tmp
```

Inspect a specific log:

```bash
kubectl -n <NAMESPACE> exec -c datahub-gms datahub-k8s-0 -- cat /tmp/<LOG_FILE>
```

## Slow startup is normal

The GMS is a large JVM application; a cold start can take several minutes, and the first-ever start also runs a one-time system upgrade job. The charm's health checks are sized for this: Pebble restarts a workload only after roughly five minutes of consecutive failed checks. Give a fresh deployment up to 30 minutes before digging deeper.

## The frontend loads but the page is blank

A blank page with browser console errors about `/assets/...` MIME types means the frontend is served behind a path-prefix ingress route. The DataHub SPA requires host-based routing; see [Expose DataHub with ingress](expose-with-ingress.md).

## Getting more help

- [File a bug](https://github.com/canonical/datahub-k8s-operator/issues)
- [Ask on the Charmhub Discourse](https://discourse.charmhub.io/)
