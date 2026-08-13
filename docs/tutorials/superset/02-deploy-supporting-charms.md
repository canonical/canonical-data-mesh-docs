(tutorial-superset-deploy-supporting-charms)=

# Deploy supporting charms

Superset needs two backing services before it can start: a PostgreSQL database for its metadata and a Redis instance for caching and task queuing. Both are deployed as charms in the same model.

## Deploy PostgreSQL

Superset stores its metadata, such as users, database connections, datasets, and dashboard definitions, in [Charmed PostgreSQL](https://charmhub.io/postgresql-k8s):

```bash
juju deploy postgresql-k8s --channel 14/stable --trust
```

## Deploy Redis

[Charmed Redis](https://charmhub.io/redis-k8s) serves as both the results cache and the message broker that connects the Superset web server to its Celery workers:

```bash
juju deploy redis-k8s --channel latest/edge --trust
```

## Wait for both charms to settle

Watch the deployment until both applications and units report `active`:

```bash
juju status --watch 2s
```

```text
Model              Controller           Cloud/Region        Version  SLA          Timestamp
superset-tutorial  superset-controller  microk8s/localhost  3.6.12   unsupported  10:51:29+01:00

App             Version  Status  Scale  Charm           Channel      Rev  Address         Exposed  Message
postgresql-k8s  14.15    active      1  postgresql-k8s  14/stable    495  10.152.183.243  no
redis-k8s       7.2.5    active      1  redis-k8s       latest/edge   36  10.152.183.182  no

Unit               Workload  Agent  Address      Ports  Message
postgresql-k8s/0*  active    idle   10.1.255.10         Primary
redis-k8s/0*       active    idle   10.1.255.21
```

```{note}
The PostgreSQL deployment can take around 10 minutes to reach `active`. Units
may pass through `waiting` or `blocked` states while they settle.
```

With the backing services running, continue to {ref}`deploy Superset <tutorial-superset-deploy-superset>`.
