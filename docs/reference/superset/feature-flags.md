(reference-superset-feature-flags)=

# Supported feature flags

Superset gates optional functionality behind [feature flags](https://superset.apache.org/docs/configuration/feature-flags/). The charm exposes them through the `feature-flags` configuration option, which takes a comma-separated list. Prefix a flag with `!` to disable one that Superset enables by default:

```bash
juju config superset-k8s feature-flags=DASHBOARD_RBAC,ESTIMATE_QUERY_COST,!DRILL_BY
```

The option is a complete list, not an addition: setting it replaces the previous value, and flags left out fall back to Superset's own defaults. The charm rejects any flag not in the table below and blocks with an error naming the unsupported flags. Apply the same list to the UI, worker, and beat applications of a deployment.

## Flags

Grouped by upstream maturity. Refer to the [Superset documentation](https://superset.apache.org/docs/configuration/feature-flags/) for what each flag does and for its default state.

| Maturity | Flags |
|---|---|
| In development | `ALERT_REPORT_TABS`, `CHART_PLUGINS_EXPERIMENTAL`, `DATE_RANGE_TIMESHIFTS_ENABLED`, `ENABLE_ADVANCED_DATA_TYPES`, `PRESTO_EXPAND_DATA`, `SHARE_QUERIES_VIA_KV_STORE`, `TAGGING_SYSTEM` |
| In testing | `ALERT_REPORTS`, `ALLOW_FULL_CSV_EXPORT`, `CACHE_IMPERSONATION`, `CONFIRM_DASHBOARD_DIFF`, `DATE_FORMAT_IN_EMAIL_SUBJECT`, `DYNAMIC_PLUGINS`, `ENABLE_SUPERSET_META_DB`, `ESTIMATE_QUERY_COST`, `GLOBAL_ASYNC_QUERIES`, `IMPERSONATE_WITH_EMAIL_PREFIX`, `PLAYWRIGHT_REPORTS_AND_THUMBNAILS`, `RLS_IN_SQLLAB`, `SSH_TUNNELING`, `USE_ANALOGOUS_COLORS` |
| Stable, launch or deprecation path | `DASHBOARD_VIRTUALIZATION` |
| Stable, runtime configuration | `ALERTS_ATTACH_REPORTS`, `ALLOW_ADHOC_SUBQUERY`, `DASHBOARD_RBAC`, `DATAPANEL_CLOSED_BY_DEFAULT`, `DRILL_BY`, `DRUID_JOINS`, `EMBEDDABLE_CHARTS`, `EMBEDDED_SUPERSET`, `ENABLE_TEMPLATE_PROCESSING`, `ESCAPE_MARKDOWN_HTML`, `LISTVIEWS_DEFAULT_CARD_VIEW`, `SCHEDULED_QUERIES`, `SLACK_ENABLE_AVATARS`, `SQL_VALIDATORS_BY_ENGINE`, `SQLLAB_BACKEND_PERSISTENCE`, `THUMBNAILS` |
| Deprecated | `AVOID_COLORS_COLLISION`, `DRILL_TO_DETAIL`, `ENABLE_JAVASCRIPT_CONTROLS`, `KV_STORE` |

## Flags with charm-specific behavior

| Flag | Behavior |
|---|---|
| `ALERT_REPORTS` | Enables email alerts and reports, and registers the beat schedule that dispatches them. Also turns on `PLAYWRIGHT_REPORTS_AND_THUMBNAILS` automatically, so do not set that flag yourself. Requires the SMTP secret; see {ref}`Enable alerts and reports <how-to-superset-enable-alerts-and-reports>`. |
| `GLOBAL_ASYNC_QUERIES` | Runs queries through the worker applications and delivers results over an asynchronous event channel backed by Redis. Requires at least one worker; see {ref}`Scale and tune performance <how-to-superset-scale-and-tune-performance>`. |
