(reference-superset-index)=

# Superset reference

**Superset** ([superset-k8s](https://charmhub.io/superset-k8s)) is a Kubernetes operator for [Apache Superset](https://superset.apache.org/), the data exploration and visualization platform that provides the Canonical Data Mesh with SQL exploration, charts, and dashboards. A deployment is made of one to three applications of the same charm - a web server, asynchronous workers, and a beat scheduler - backed by PostgreSQL and Redis, and integrating with Trino, ingress providers, and the Canonical Observability Stack.

## Reference pages

```{toctree}
:maxdepth: 1

system-requirements
integrations
feature-flags
observability
versions-and-channels
```

## Generated reference on Charmhub

Configuration options, actions, and relation endpoints are generated from the charm itself and published on Charmhub, they are not duplicated here:

- [Configuration options](https://charmhub.io/superset-k8s/configurations)
- [Actions](https://charmhub.io/superset-k8s/actions)
- [Integrations](https://charmhub.io/superset-k8s/integrations)

## Source and issues

- [Source repository](https://github.com/canonical/superset-k8s-operator)
- [Report a bug](https://github.com/canonical/superset-k8s-operator/issues)
