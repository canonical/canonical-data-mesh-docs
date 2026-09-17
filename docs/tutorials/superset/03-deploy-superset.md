(tutorial-superset-deploy-superset)=

# Deploy Superset

With PostgreSQL and Redis running, you can now deploy Superset itself and connect it to both services.

## Deploy the charm

A Superset deployment is made up of three functions, each deployed as its own application from the same charm and selected with the `charm-function` configuration option:

- the web server and UI (`app-gunicorn`, the default),
- the worker that runs asynchronous queries and scheduled reports (`worker`),
- the beat scheduler that triggers periodic tasks (`beat`).

This tutorial deploys the web server only. To add workers and a scheduler later, see {ref}`Scale and tune performance <how-to-superset-scale-and-tune-performance>`.

Superset needs two signing keys: `secret-key`, which signs session cookies and encrypts the database connection passwords stored in its metadata database, and `async-queries-jwt`, which signs the cookie used for asynchronous query results. Generate them and store them in a Juju user secret:

```bash
juju add-secret superset-signing-keys \
  secret-key="$(openssl rand -base64 42)" \
  async-queries-jwt="$(openssl rand -hex 32)"
```

The command prints the secret ID (a URI starting with `secret:`). Note it down; the next step passes it to the charm.

Deploy the charm, grant it access to the secret, then point it at the secret:

```bash
juju deploy superset-k8s --channel 6/stable
juju grant-secret superset-signing-keys superset-k8s
juju config superset-k8s signing-keys-secret-id=<SECRET_ID>
```

Grant the secret before setting the option. Granting does not notify the charm, so done the other way round the unit keeps reporting that it cannot access the secret until its next periodic status check.

```{caution}
Keep `secret-key` stable for the lifetime of a deployment: it encrypts the
database connection passwords held in the metadata database, so changing it
makes every stored connection password unreadable. Keep the secret backed up
independently of the model and treat it as part of a Superset backup.
```

The admin account is created on first start with a password the charm generates.
The signing keys are the only credentials you supply.

The charm channel selects the Superset major version, for example `6/stable` for Superset 6. See {ref}`Versions and channels <reference-superset-versions-and-channels>` for the full list.

Until it is integrated with its backing services, the charm reports `blocked` and names every relation it is missing, for example `Required relations missing: PostgreSQL, Redis`.

## Integrate with PostgreSQL and Redis

```bash
juju integrate superset-k8s postgresql-k8s
juju integrate superset-k8s redis-k8s
```

Superset initializes its database schema and creates the admin account on first start, which takes a few minutes. Watch the deployment until the unit reports `active` with the message `Status check: UP`:

```bash
juju status --relations --watch 2s
```

```text
App             Version  Status  Scale  Charm           Channel    Rev  Address         Exposed  Message
postgresql-k8s  14.15    active      1  postgresql-k8s  14/stable  495  10.152.183.243  no
redis-k8s       7.2.5    active      1  redis-k8s       latest/edge 36  10.152.183.182  no
superset-k8s    v6.1.0   active      1  superset-k8s    6/stable    52  10.152.183.135  no       Status check: UP

Unit               Workload  Agent  Address      Ports  Message
postgresql-k8s/0*  active    idle   10.1.255.10         Primary
redis-k8s/0*       active    idle   10.1.255.21
superset-k8s/0*    active    idle   10.1.255.5          Status check: UP

Integration provider     Requirer                    Interface          Type     Message
postgresql-k8s:database  superset-k8s:postgresql_db  postgresql_client  regular
redis-k8s:redis          superset-k8s:redis          redis              regular
superset-k8s:peer        superset-k8s:peer           superset           peer
```

## Log in to the UI

Find the unit's IP address in the `Address` column of the `superset-k8s/0` unit, then open Superset in your browser:

```text
http://<UNIT_ADDRESS>:8088
```

With the Multipass setup, that address is only routable from inside the VM. 
Either browse from `multipass shell superset-tutorial`, or forward the port out of the VM:

```shell
# Run inside the VM. `multipass list` gives the VM address to then browse from the host.
microk8s kubectl port-forward -n superset-tutorial pod/superset-k8s-0 8088:8088 --address 0.0.0.0
```

Log in with the username `admin`. The charm generates the password; read it with:

```bash
juju run superset-k8s/leader get-admin-password
```

```{note}
Accessing the unit IP directly is fine for a local tutorial. For anything
beyond that, expose Superset through an ingress with TLS - see
{ref}`Expose Superset with ingress <how-to-superset-expose-with-ingress>`.
```

Superset is up and running. Continue to {ref}`create a dashboard <tutorial-superset-create-a-dashboard>`.
