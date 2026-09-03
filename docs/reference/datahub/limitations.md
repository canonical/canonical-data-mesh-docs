(reference-datahub-limitations)=

# Limitations

This page collects what a charmed DataHub deployment cannot do. Everything here is a property of the charm as it is published, not of DataHub itself.

## One pod, three workloads

The charm manages a single pod that runs the GMS, the frontend and the actions framework, and the frontend and actions containers reach the GMS over `localhost`. The three cannot be placed, sized or scaled independently of each other: adding a unit adds a replica of all three.

## Horizontal scaling and the session store

Scaling out works with the default session handling, which keeps the user session in OIDC cookies. Setting `use-play-cache-session-store` to `true` moves sessions into the frontend process, which makes the frontend stateful: a request routed to another unit lands on a unit that does not know the session. Use that option only on a single-unit deployment. It exists because the cookie-based store produces large response headers that some reverse proxy buffers reject.

## Sharing backing services between deployments

Kafka topics and OpenSearch indices can be given a prefix with `kafka-topic-prefix` and `opensearch-index-prefix`, so several DataHub deployments can share one Kafka or one OpenSearch. The metadata database name is fixed at `datahub_db` and has no equivalent option, so two deployments related to the same PostgreSQL application would end up in the same database. Give each deployment its own PostgreSQL application.

## The encryption keys are supplied, not generated

`encryption-keys-secret-id` is required at deploy time and the charm never generates the keys itself. `gms-key` is the key DataHub encrypts stored secrets with, such as the passwords behind ingestion sources, so it belongs to the data rather than to the deployment: a deployment rebuilt against surviving backends must be given the same key, or those secrets cannot be decrypted. Keep the Juju user secret backed up independently of the model, and treat it as part of a DataHub backup. See {ref}`Back up and restore <how-to-datahub-back-up-and-restore>`.

## Single sign-on requires a TLS-terminated frontend ingress

The charm refuses to configure OIDC against a plain HTTP URL and blocks with `OIDC requires an HTTPS ingress URL` until the frontend ingress publishes an `https` URL. See {ref}`Enable single sign-on <how-to-datahub-enable-sso>`.

## The frontend has to be served at the root of a hostname

The frontend is a single-page application compiled with absolute asset paths, so it must be served at the root of a hostname rather than under a path prefix. With Traefik the charm asks for the per-application prefix to be stripped; another ingress provider has to be configured to route the root path.
