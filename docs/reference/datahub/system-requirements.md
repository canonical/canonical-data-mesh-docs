# System requirements

## Software

| Requirement | Version |
|---|---|
| Juju | 3.4 or later |
| Kubernetes | A cluster supported by Juju, with a default storage class |
| DataHub workload | 1.4 series (packaged as rocks with the charm resources) |

## Cloud topology

DataHub requires a multi-cloud setup:

| Component | Cloud |
|---|---|
| DataHub (this charm) | Kubernetes |
| PostgreSQL | Kubernetes or machine |
| Kafka and ZooKeeper | Kubernetes or machine |
| OpenSearch | Machine only |

A multi-cloud setup can be achieved with a single controller that has multiple clouds registered (used by the Terraform product module) or with two controllers of one cloud each. The single-controller setup uses cross-model relations, which are more stable than cross-controller relations.

## Host requirements for OpenSearch

The machine host running OpenSearch requires these kernel parameters:

```text
vm.max_map_count=262144
vm.swappiness=0
net.ipv4.tcp_retries2=5
fs.file-max=1048576
```

## Resource sizing

For a single-host evaluation deployment (the [tutorial](../../tutorials/datahub/index.md) setup with the full data platform co-located):

| Resource | Minimum |
|---|---|
| CPU | 8 threads |
| RAM | 24 GB |
| Disk | 80 GB |

The DataHub pod alone runs three workloads: two JVM services (GMS and frontend) and the Python-based actions framework. Production sizing depends on metadata volume and ingestion frequency; monitor JVM heap metrics (see [Observability](observability.md)) to size the deployment.

## Supported dependency channels

The deployment flows in this documentation are tested with:

| Charm | Channel |
|---|---|
| postgresql | 14/stable |
| kafka | 3/stable |
| zookeeper | 3/stable |
| opensearch | 2/stable |
| self-signed-certificates | latest/stable |
| traefik-k8s | latest/stable |
