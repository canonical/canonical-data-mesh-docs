(how-to-superset-trust-a-certificate-authority)=

# Trust a certificate authority

{ref}`Exposing Superset with ingress <how-to-superset-expose-with-ingress>` secures inbound traffic to the user interface. This guide covers the opposite direction: letting Superset trust the certificate authority (CA) of the data sources it queries, such as a TLS-enabled Kyuubi or Hive Thrift endpoint.

## Relate a certificate provider

Relate Superset to any charm providing the `tls-certificates` interface, such as [self-signed-certificates](https://charmhub.io/self-signed-certificates):

```bash
juju deploy self-signed-certificates --channel 1/stable
juju integrate superset-k8s:certificates self-signed-certificates:certificates
```

Outside of testing, use a production certificate provider in its place.

The charm installs the provider's CA into the workload's system trust store and restarts the application, so most drivers validate the data source certificate with no further configuration.

## Point a driver at the CA file

Some drivers do not read the system trust store and need an explicit CA file. The same CA is written to a stable path inside the container for that case:

```text
/etc/ssl/certs/charm-ca.pem
```

Reference it from the **Advanced** > **Other** > **Engine Parameters** field of the Superset database connection:

```json
{
  "connect_args": {
    "ssl_cert": "/etc/ssl/certs/charm-ca.pem"
  }
}
```

```{note}
The path is managed by the charm. It is re-created whenever the certificate is renewed or the pod is respawned, and removed when the `certificates` relation is broken. Treat it as read-only.
```

## Verify

Confirm the CA is present in the workload container:

```bash
juju ssh --container superset superset-k8s/0 ls -l /etc/ssl/certs/charm-ca.pem
```

Then open the data source in **SQL Lab** and run a trivial query. A handshake failure surfaces as an SSL error in the query result rather than a connection timeout.
