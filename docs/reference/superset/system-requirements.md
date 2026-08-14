(reference-superset-system-requirements)=

# System requirements

## Software

| Requirement | Version |
|---|---|
| Juju | 3.4 or later |
| Kubernetes | A cluster supported by Juju, with a default storage class |
| Superset workload | Selected by the charm track, for example `6/stable` for the Superset 6 series (packaged as a [rock](https://documentation.ubuntu.com/rockcraft/en/stable/) and attached as the `superset-image` resource) |

Superset and its backing services all run on Kubernetes, so a single Kubernetes cloud is enough.

## Resource sizing

For a single-host evaluation deployment (the {ref}`tutorial <tutorial-superset-index>` setup, with PostgreSQL and Redis co-located):

| Resource | Minimum |
|---|---|
| CPU | 2 threads, 4 recommended |
| RAM | 8 GB |
| Disk | 40 GB |

Production sizing depends on the number of concurrent users and the query mix. Scale the UI and worker applications horizontally rather than growing a single pod, and size PostgreSQL for the metadata database and Redis for the results cache accordingly. See {ref}`Scale and tune performance <how-to-superset-scale-and-tune-performance>`.

## Supported dependency channels

The deployment flows in this documentation are tested with:

| Charm | Channel | Required |
|---|---|---|
| [postgresql-k8s](https://charmhub.io/postgresql-k8s) | 14/stable | Yes |
| [redis-k8s](https://charmhub.io/redis-k8s) | latest/edge | Yes |
| [nginx-ingress-integrator](https://charmhub.io/nginx-ingress-integrator) | latest/stable | For ingress and TLS |
| [trino-k8s](https://charmhub.io/trino-k8s) | latest/edge | For the Trino integration |
| [cos-lite](https://charmhub.io/cos-lite) | latest/stable | For observability |

## Ports

| Port | Protocol | Purpose |
|---|---|---|
| 8088 | TCP | Superset web server and API |
| 9102 | TCP | Prometheus metrics |
| 9125 | UDP | StatsD metrics ingest (in-pod) |
