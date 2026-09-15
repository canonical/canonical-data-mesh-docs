(how-to-superset-enable-alerts-and-reports)=

# Enable alerts and reports

This guide describes how to enable Superset's email alerts and reports, which send dashboard and chart screenshots on a schedule or when an alert condition is met.

## Prerequisites

Alerts and reports need all three Superset functions running, because the beat scheduler triggers the task and a worker renders the screenshot:

- a UI application (`charm-function=app-gunicorn`),
- a worker application (`charm-function=worker`),
- a beat application (`charm-function=beat`).

See {ref}`Scale and tune performance <how-to-superset-scale-and-tune-performance>` for how to deploy the worker and beat applications.

## Deploy an SMTP integrator

The outgoing mail relay reaches Superset over the `smtp` relation, provided by the [smtp-integrator](https://charmhub.io/smtp-integrator) charm. The relay password is kept in a Juju secret, so it never appears in configuration or on the relation.

Store the password in a Juju user secret. The key must be `password`:

```bash
juju add-secret smtp-password password=<SMTP_PASSWORD>
```

The command prints the secret ID. Deploy the integrator with the relay details:

```bash
juju deploy smtp-integrator --channel latest/stable \
  --config host=<SMTP_HOST> \
  --config port=<SMTP_PORT> \
  --config auth_type=plain \
  --config user=<SMTP_USERNAME> \
  --config transport_security=starttls \
  --config smtp_sender=<SENDER_ADDRESS>
```

`auth_type` is one of `none`, `not_provided` or `plain`, and `transport_security` is one of `none`, `starttls` or `tls`. Set `skip_ssl_verify=true` only if the relay presents a certificate you cannot validate. `smtp_sender` is the address Superset sends from and is required: without it the Superset applications block with `the smtp relation is missing smtp_sender`.

The integrator publishes the ID of the secret on the relation and each application reads the password from it, so grant the secret to the integrator and to every Superset application, then point the integrator at it:

```bash
juju grant-secret smtp-password smtp-integrator,superset-k8s,superset-k8s-worker,superset-k8s-beat
juju config smtp-integrator password_secret=<SECRET_ID>
```

An application that has not been granted the secret blocks with `smtp relation data is unusable: Could not consume secret <SECRET_ID>`. To change the password later, update the secret with `juju update-secret smtp-password password=<NEW_PASSWORD>`; the applications pick up the new value without being reconfigured.

If the relay needs no authentication, set `auth_type=none` and skip the secret.

Relate the integrator to all three applications:

```bash
juju integrate superset-k8s:smtp smtp-integrator
juju integrate superset-k8s-worker:smtp smtp-integrator
juju integrate superset-k8s-beat:smtp smtp-integrator
```

## Configure the applications

Each application needs the `ALERT_REPORTS` feature flag. The worker also needs a `server-alias` that resolves to the UI and an `external-url` for the links in the emails:

```bash
juju config superset-k8s feature-flags=ALERT_REPORTS

juju config superset-k8s-worker \
  feature-flags=ALERT_REPORTS \
  server-alias=superset-k8s \
  external-url=https://<YOUR_HOSTNAME>

juju config superset-k8s-beat \
  feature-flags=ALERT_REPORTS
```

```{important}
`feature-flags` is a complete list, not an addition. Include the flags already
set on the application along with `ALERT_REPORTS`, for example
`feature-flags=GLOBAL_ASYNC_QUERIES,ALERT_REPORTS`.
```

Setting `ALERT_REPORTS` without the `smtp` relation blocks the application with `ALERT_REPORTS requires an smtp relation`, since reports would render and then fail to deliver. The block clears as soon as the relation is added, so the two steps can be done in either order.

`server-alias` is the hostname the worker's browser loads dashboards from, so it must be a name a worker pod resolves to the UI application. That is typically the UI application's name, since Kubernetes provides in-namespace DNS for it, and it is not the default when the UI is deployed under any other name. For cross-namespace setups, use the fully qualified service name; see the [Kubernetes DNS documentation](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/).

`external-url` is the URL recipients open from the email, so it must be a URL their browser can reach. The charm does not derive it from anything: the worker builds the link and holds no ingress relation, so set it there even when the UI is exposed through an ingress provider. Left unset, the links in the email come out relative and recipients cannot open them.

Two more worker options shape the emails. `email-subject-prefix` changes the `[Superset] ` prefix on every alert and report subject line. `screenshot-timeout` is how long, in seconds, the worker waits for a dashboard or chart to render before giving up on the screenshot; the default is 600, so raise it only if large dashboards still time out.

## Verify

In the Superset UI, go to **Settings** > **Alerts & reports** and create a report on any dashboard with a one-off schedule. The email arrives with a screenshot attached, and the links in it point at `external-url`.

To exercise the rendering pipeline without sending mail, set `report-dry-run=true` on all three applications. Reports then render and log `ALERT_REPORTS_NOTIFICATION_DRY_RUN is enabled` instead of being delivered, and the `smtp` relation is not required while it is set.

## How screenshots are rendered

Since Superset 6, report and alert screenshots are rendered by Playwright driving a headless Chromium browser, replacing the Selenium and Firefox stack used in Superset 5. Chromium is bundled in the charm's workload image, so no additional configuration is needed.

Enabling `ALERT_REPORTS` automatically turns on the `PLAYWRIGHT_REPORTS_AND_THUMBNAILS` feature flag that selects the Playwright renderer. Do not set that flag yourself through `feature-flags`.
