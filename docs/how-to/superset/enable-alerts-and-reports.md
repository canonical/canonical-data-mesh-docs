(how-to-superset-enable-alerts-and-reports)=

# Enable alerts and reports

This guide describes how to enable Superset's email alerts and reports, which send dashboard and chart screenshots on a schedule or when an alert condition is met.

## Prerequisites

Alerts and reports need all three Superset functions running, because the beat scheduler triggers the task and a worker renders the screenshot:

- a UI application (`charm-function=app-gunicorn`),
- a worker application (`charm-function=worker`),
- a beat application (`charm-function=beat`).

See {ref}`Scale and tune performance <how-to-superset-scale-and-tune-performance>` for how to deploy the worker and beat applications.

## Create the SMTP secret

Put the SMTP credentials in a file. All keys except `email-subject-prefix` are required:

```yaml
# smtp.yaml
host: <SMTP_HOST>
port: <SMTP_PORT>
username: <SMTP_USERNAME>
password: <SMTP_PASSWORD>
email: <SENDER_ADDRESS>
email-subject-prefix: "[Superset] "
superset-external-url: https://<YOUR_HOSTNAME>
ssl: <TRUE_OR_FALSE>
starttls: <TRUE_OR_FALSE>
ssl-server-auth: <TRUE_OR_FALSE>
```

`superset-external-url` is the URL used to build the links in the emails, so it must be a URL your recipients can open in a browser. Set `ssl-server-auth` to `true` when the SMTP server presents a valid certificate. The remaining keys mirror the email settings in Superset's [alerts and reports documentation](https://superset.apache.org/docs/configuration/alerts-reports/).

Create a Juju secret from the file and grant it to all three applications:

```bash
juju add-secret smtp-config --file=smtp.yaml
```

The command prints a secret URI. Grant it to each application:

```bash
juju grant-secret smtp-config superset-k8s
juju grant-secret smtp-config superset-k8s-worker
juju grant-secret smtp-config superset-k8s-beat
```

## Configure the applications

Each application needs the `ALERT_REPORTS` feature flag, the secret ID, and a `server-alias` that resolves to the UI:

```bash
juju config superset-k8s \
  feature-flags=ALERT_REPORTS \
  server-alias=superset-k8s \
  smtp-secret-id=<SECRET_ID>
```

Repeat for `superset-k8s-worker` and `superset-k8s-beat`.

```{important}
`feature-flags` is a complete list, not an addition. Include the flags already
set on the application along with `ALERT_REPORTS`, for example
`feature-flags=GLOBAL_ASYNC_QUERIES,ALERT_REPORTS`.
```

Workers render reports by browsing the UI over HTTP, so `server-alias` must be a name a worker pod can resolve to the UI application - typically the UI application's name, since Kubernetes provides in-namespace DNS for it. For cross-namespace setups, use the fully qualified service name; see the [Kubernetes DNS documentation](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/).

## Verify

In the Superset UI, go to **Settings** > **Alerts & reports** and create a report on any dashboard with a one-off schedule. The email arrives with a screenshot attached, and the links in it point at `superset-external-url`.

## How screenshots are rendered

Since Superset 6, report and alert screenshots are rendered by Playwright driving a headless Chromium browser, replacing the Selenium and Firefox stack used in Superset 5. Chromium is bundled in the charm's workload image, so no additional configuration is needed.

Enabling `ALERT_REPORTS` automatically turns on the `PLAYWRIGHT_REPORTS_AND_THUMBNAILS` feature flag that selects the Playwright renderer. Do not set that flag yourself through `feature-flags`.
