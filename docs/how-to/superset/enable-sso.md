(how-to-superset-enable-sso)=

# Enable single sign-on

This guide describes how to let users log in to Superset with their Google accounts instead of local Superset credentials, and how to control the role they receive on first login.

## Prerequisites

- Superset is served over HTTPS on a stable hostname. See {ref}`Expose Superset with ingress <how-to-superset-expose-with-ingress>`.
- You have access to a [Google Cloud project](https://console.cloud.google.com/projectcreate).

## Obtain OAuth 2.0 credentials

1. Go to the [Google Cloud credentials page](https://console.cloud.google.com/apis/credentials).
2. Select **+ Create credentials**, then **OAuth client ID**.
3. Choose **Web application** as the application type and give it a name.
4. Under **Authorized redirect URIs**, add `https://<YOUR_HOSTNAME>/oauth-authorized/google`.
5. Create the client, then copy the client ID and client secret.

## Configure the charm

Write the credentials to a configuration file:

```yaml
# oauth.yaml
superset-k8s:
  google-client-id: <CLIENT_ID>
  google-client-secret: <CLIENT_SECRET>
  oauth-domain: <COMPANY_DOMAIN>
  oauth-admin-email: <ADMIN_EMAIL>
```

`oauth-domain` restricts authentication to accounts in that domain, for example `canonical.com`. `oauth-admin-email` takes one email address or a comma-separated list; those users are given the `Admin` role on initialization.

Apply the file:

```bash
juju config superset-k8s --file=oauth.yaml
```

Apply the same configuration to your worker and beat applications if you run them, so that they share the same view of user identities.

## Choose the self-registration role

A Superset account is created automatically the first time a user authenticates. By default the account receives Superset's least privileged role, `Public`, and an administrator can elevate it afterwards in the UI or through the API.

To grant a different role on self-registration:

```bash
juju config superset-k8s self-registration-role=Gamma
```

The role must already exist in Superset and the value is case-sensitive; the charm validates it against the roles in the metadata database and blocks on an unknown value.

```{note}
Roles created by the {ref}`Trino integration <how-to-superset-integrate-with-trino>`
matter here: database access permissions for Trino catalogs are granted to the
role named by `self-registration-role`, so every self-registered user inherits
access to the catalogs Superset manages.
```

## Verify

Open Superset in a private browser window. You are redirected to Google, and after authenticating you land in Superset as the Google account. Check the created account under **Settings** > **List users**.
