(reference-superset-versions-and-channels)=

# Versions and channels

The charm follows the major version of Apache Superset: each Charmhub track carries one Superset major series, and each risk level within a track carries a release stage.

| Track | Superset series |
|---|---|
| `6` | Superset 6 |
| `5` | Superset 5 |
| `latest` | Development builds from the `main` branch; not tied to a series |

Deploy from a versioned track to pin a major version:

```bash
juju deploy superset-k8s --channel 6/stable
```

Upgrading between major stable tracks is a supported, non-breaking operation:

```bash
juju refresh superset-k8s --channel 6/stable
```

## Charm and workload compatibility

The workload image is attached to the charm revision as the `superset-image` resource, so deploying from any `*/stable` channel gives a matching charm and workload pair. Pinning a revision by hand is only necessary in the rare case of reproducing an older deployment:

```bash
juju deploy superset-k8s --channel latest/stable --revision <REVISION> --resource superset-image=<RESOURCE_REVISION>
```

Charmhub's [releases page](https://charmhub.io/superset-k8s) lists which resource revision belongs to each charm revision.

## Branches and channels

Development follows one branch per track in the [charm repository](https://github.com/canonical/superset-k8s-operator):

| Branch | Channel |
|---|---|
| `main` | `latest/edge` |
| `track/<N>` | `<N>/edge` |

Releases are promoted from `<N>/edge` to `<N>/stable` after validation.
