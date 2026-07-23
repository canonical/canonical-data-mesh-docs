(tutorial-airbyte-run-a-sync)=

# Run your first sync

A running Airbyte isn't much use until it moves data. In this step you open the Airbyte UI and use Airbyte's built-in test connectors - a **Sample Data** source and an **End-to-End Testing** destination - to create a connection and run your first sync, without needing any external database or API.

## Open the Airbyte UI

The `airbyte-server` container serves the web UI on port `8001`. Find the unit's IP address in the output of `juju status` (the `Address` column of the `airbyte-k8s/0` unit), then open Airbyte in your browser:

```text
http://<UNIT_ADDRESS>:8001
```

```{note}
Accessing the unit IP directly is fine for a local tutorial. For anything
beyond that, put Airbyte behind an ingress with TLS and authentication - see
{ref}`Secure Airbyte deployments <how-to-airbyte-secure-deployments>`.
```

## Set up a test source

The **Sample Data (Faker)** source generates realistic-looking records in memory, so it needs no external system to connect to.

1. Select **Sources** in the left-hand navigation, then click **New source**.
2. Search for `Sample Data` and select the **Sample Data (Faker)** connector.
3. Leave the defaults (for example, a **Count** of 1000 records) and click **Set up source**.

## Set up a test destination

The **End-to-End Testing** destination discards everything it receives, so no object storage or warehouse is required to complete a sync.

1. Select **Destinations** in the left-hand navigation, then click **New destination**.
2. Search for `End-to-End Testing` and select the **End-to-End Testing (/dev/null)** connector.
3. Keep the default logging mode and click **Set up destination**.

## Create a connection and sync

1. Select **Connections** in the left-hand navigation, then click **New connection**.
2. Choose the **Sample Data (Faker)** source and the **End-to-End Testing** destination you just created.
3. On the connection configuration screen, keep the default replication frequency and streams, then click **Set up connection**.
4. Click **Sync now** to trigger the first run.

The sync appears in the connection's **Job History** as **Running**, then **Succeeded** after a few moments. The job summary reports the number of records emitted and committed, confirming that Airbyte extracted data from the source and delivered it to the destination through Temporal.

You now have a working Charmed Airbyte deployment and have run a sync end to end. The same flow applies to real connectors: swap the test source and destination for any of the hundreds in Airbyte's catalog.

Continue to {ref}`clean up <tutorial-airbyte-cleanup>`.
