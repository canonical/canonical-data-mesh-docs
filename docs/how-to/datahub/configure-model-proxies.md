(how-to-datahub-configure-model-proxies)=

# Configure model proxies

If your model runs in a restricted network, configure Juju model proxies so the DataHub containers can reach external services, such as Python package indexes for ingestion recipes or an external identity provider.

## Set the proxy configuration

```bash
juju model-config juju-http-proxy=http://proxy.example:8080
juju model-config juju-https-proxy=http://proxy.example:8080
juju model-config juju-no-proxy=127.0.0.1,localhost,.svc,.cluster.local
```

The charm propagates the settings as follows:

- `datahub-actions` receives the standard proxy variables (`HTTP_PROXY`, `HTTPS_PROXY`, `NO_PROXY`, and lowercase variants), which are used by Python tooling and ingestion runs.
- `datahub-frontend` receives the equivalent JVM proxy settings, used for example when contacting an identity provider.
- Trino ingestion sources managed by the charm get proxy variables injected into their recipes.

## Apply the change

After changing the model proxy settings, allow one `update-status` cycle or trigger another hook event (for example a configuration change) so the charm refreshes its Pebble plans.

To verify the variables on a container:

```bash
kubectl -n <NAMESPACE> exec -c datahub-actions datahub-k8s-0 -- pebble plan | grep -i proxy
```
