---
title: "Known issues: Azure IoT Operations"
description: Known issues for the MQTT broker, Layered Network Management (preview), connector for OPC UA, OPC PLC simulator, data flows, and operations experience web UI.
author: dominicbetts
ms.author: dobett
ms.topic: troubleshooting-known-issue
ms.date: 04/16/2025
---

# Known issues: Azure IoT Operations

This article lists the current known issues for Azure IoT Operations.

## Deploy, update, and uninstall issues

This section lists current known issues that might occur when you deploy, update, or uninstall Azure IoT Operations.

### Error creating custom resources

---

Issue ID: 9091

---

Log signature: `"code": "ExtensionOperationFailed", "message": "The extension operation failed with the following error:  Error occurred while creating custom resources needed by system extensions"`

---

The message `Error occurred while creating custom resources needed by system extensions` indicates that your deployment failed due to a known sporadic issue.

To work around this issue, use the `az iot ops delete` command with the `--include-deps` flag to delete Azure IoT Operations from your cluster. When Azure IoT Operations and its dependencies are deleted from your cluster, retry the deployment.

### Codespaces restart error

---

Issue ID: 9941

---

Log signature: `"This codespace is currently running in recovery mode due to a configuration error."`

---

If you deploy Azure IoT Operations in GitHub Codespaces, shutting down and restarting the Codespace causes a `This codespace is currently running in recovery mode due to a configuration error` issue.

Currently, there's no workaround for the issue. If you need a cluster that supports shutting down and restarting, choose one of the options in [Prepare your Azure Arc-enabled Kubernetes cluster](../deploy-iot-ops/howto-prepare-cluster.md).

### Helm package enters a stuck state during update

---

Issue ID: 9928

---

Log signature: `"Message: Update failed for this resource, as there is a conflicting operation in progress. Please try after sometime."`

---

When you update Azure IoT Operations, the Helm package might enter a stuck state, preventing any helm install or upgrade operations from proceeding. This scenario results in the error message `Update failed for this resource, as there is a conflicting operation in progress. Please try after sometime.`, which blocks further updates.

To work around this issue, follow these steps:

1. Identify the stuck components by running the following command:

   ```sh
   helm list -n azure-iot-operations --pending
   ```

    In the output, look for the release name of components, `<component-release-name>`, which have a status of `pending-upgrade` or `pending-install`. This issue might affect the following components:

      - `-adr`
      - `-akri`
      - `-connectors`
      - `-mqttbroker`
      - `-dataflows`
      - `-schemaregistry`

1. Using the `<component-release-name>` from step 1, retrieve the revision history of the stuck release. You need to run the following command for **each component from step 1**. For example, if components `-adr` and `-mqttbroker` are stuck, you run the following command twice, once for each component:

   ```sh
   helm history <component-release-name> -n azure-iot-operations
   ```

    Make sure to replace `<component-release-name>` with the release name of the components that are stuck. In the output, look for the last revision that has a status of `Deployed` or `Superseded` and note the revision number.

1. Using the **revision number from step 2**, roll back the Helm release to the last successful revision. You need to run the following command for each component, `<component-release-name>`, and its revision number, `<revision-number>`, from steps 1 and 2.

    ```sh
    helm rollback <component-release-name> <revision-number> -n azure-iot-operations
    ```
  
    > [!IMPORTANT]
    > You need to repeat steps 2 and 3 for each component that is stuck. You reattempt the upgrade only after all components are rolled back to the last successful revision.

1. After the rollback of each component is complete, reattempt the upgrade using the following command:

   ```sh
   az iot ops update
   ```

    If you receive a message stating `Nothing to upgrade or upgrade complete`, force the upgrade by appending:

    ```sh
    az iot ops upgrade ....... --release-train stable 
    ```

## MQTT broker issues

This section lists current known issues for the MQTT broker.

### MQTT broker resources aren't visible in Azure portal

---

Issue ID: 4257

---

Log signature: N/A

---

MQTT broker resources created in your cluster using Kubernetes aren't visible in the Azure portal. This result is expected because [managing Azure IoT Operations components using Kubernetes is in preview](../deploy-iot-ops/howto-manage-update-uninstall.md#preview-manage-components-using-kubernetes-deployment-manifests), and synchronizing resources from the edge to the cloud isn't currently supported.

There's currently no workaround for this issue.

## Azure IoT Layered Network Management (preview) issues

This section lists current known issues for  Azure IoT Layered Network Management.

### Layered Network Management service doesn't get an IP address

---

Issue ID: 7864

---

Log signature: N/A

---

The Layered Network Management service doesn't get an IP address when it runs on K3S on an Ubuntu host.

To work around this issue, you reinstall K3S without the _traefik ingress controller_:

```bash
curl -sfL https://get.k3s.io | sh -s - --disable=traefik --write-kubeconfig-mode 644
```

To learn more, see [Networking | K3s](https://docs.k3s.io/networking#traefik-ingress-controller).

### CoreDNS service doesn't resolve DNS queries correctly

---

Issue ID: 7955

---

Log signature: N/A

---

DNS queries don't resolve to the expected IP address while using the [CoreDNS](../manage-layered-network/howto-configure-layered-network.md#configure-coredns) service running on the child network level.

To work around this issue, upgrade to Ubuntu 22.04 and reinstall K3S.

## Connector for OPC UA issues

This section lists current known issues for the connector for OPC UA.

### Connector pod doesn't restart after configuration change

---

Issue ID: 7518

---

Log signature: N/A

---

When you add a new asset with a new asset endpoint profile to the OPC UA broker and trigger a reconfiguration, the deployment of the `opc.tcp` pods changes to accommodate the new secret mounts for username and password. If the new mount fails for some reason, the pod doesn't restart and therefore the old flow for the correctly configured assets stops as well.

### Data spike every 2.5 hours with some OPC UA simulators

---

Issue ID: 6513

---

Log signature: Increased message volume every 2.5 hours

---

Data values spike every 2.5 hours when using particular OPC UA simulators causing CPU and memory spikes. This issue isn't seen with OPC PLC simulator used in the quickstarts. No data is lost, but you can see an increase in the volume of data published from the server to the MQTT broker.

### No message schema generated if selected nodes in a dataset reference the same complex data type definition

---

Issue ID: 7369

---

Log signature: `An item with the same key has already been added. Key: <element name of the data type>`

---

No message schema is generated if selected nodes in a dataset reference the same complex data type definition (a UDT of type struct or enum).

If you select data points (node IDs) for a dataset that share non-OPC UA namespace complex type definitions (struct or enum), then the JSON schema isn't generated. The default open schema is shown when you create a data flow instead. For example, if the data set contains three values of a data type, then whether it works or not is shown in the following table. You can substitute `int` for any OPC UA built in type or primitive type such as `string`, `double`, `float`, or `long`:

| Type of Value 1 | Type of Value 2 | Type of Value 3 | Successfully generates schema |
|-----------------|-----------------|-----------------|-----------------|
| `int` | `int` | `int` | Yes |
| `int` | `int` | `int` | Yes |
| `int` | `int` | `struct A` | Yes |
| `int` | `enum A` | `struct A` | Yes |
| `enum A` | `enum B` | `enum C` | Yes |
| `struct A` | `struct B` | `struct C` | Yes |
| `int` | `struct A` | `struct A` | No |
| `int` | `enum A` | `enum A` | No |

To work around this issue, you can either:

- Split the dataset across two or more assets.
- Manually upload a schema.
- Use the default nonschema experience in the data flow designer.

## Connector for media and connector for ONVIF issues

This section lists current known issues for the connector for media and the connector for ONVIF.

### Cleanup of unused media-connector resources

---

Issue ID: 2142

---

Log signature: N/A

---

If you delete all the `Microsoft.Media` asset endpoint profiles, the deployment for media processing isn't deleted.

To work around this issue, run the following command using the full name of your media connector deployment:

```bash
kubectl delete deployment aio-opc-media-... -n azure-iot-operations
```

### Cleanup of unused onvif-connector resources

---

Issue ID: 3322

---

Log signature: N/A

---

If you delete all the `Microsoft.Onvif` asset endpoint profiles, the deployment for media processing isn't deleted.

To work around this issue, run the following command using the full name of your ONVIF connector deployment:

```bash
kubectl delete deployment aio-opc-onvif-... -n azure-iot-operations
```

### AssetType CRD removal process doesn't complete

---

Issue ID: 6065

---

Log signature: `"Error HelmUninstallUnknown: Helm encountered an error while attempting to uninstall the release aio-118117837-connectors in the namespace azure-iot-operations. (caused by: Unknown: 1 error occurred: * timed out waiting for the condition"`

---

Sometimes, when you attempt to uninstall Azure IoT Operations from the cluster, the system can get to a state where CRD removal job is stuck in pending state and that blocks the cleanup of Azure IoT Operations.

To work around this issue, complete the following steps to manually delete the CRD and finish the uninstall:

1. Delete the AssetType CRD manually: `kubectl delete crd assettypes.opcuabroker.iotoperations.azure.com --ignore-not-found=true`

1. Delete the job definition: `kubectl delete job aio-opc-delete-crds-job-<version> -n azure-iot-operations`

1. Find the Helm release for the connectors, it's the one with `-connectors` suffix: `helm ls -a -n azure-iot-operations`

1. Uninstall Helm release without running the hook: `helm uninstall aio-<id>-connectors -n azure-iot-operations --no-hooks`

## Data flows issues

This section lists current known issues for data flows.

### Data flow resources aren't visible in the operations experience web UI

---

Issue ID: 8724

---

Log signature: N/A

---

Data flow custom resources created in your cluster using Kubernetes aren't visible in the operations experience web UI. This result is expected because [managing Azure IoT Operations components using Kubernetes is in preview](../deploy-iot-ops/howto-manage-update-uninstall.md#preview-manage-components-using-kubernetes-deployment-manifests), and synchronizing resources from the edge to the cloud isn't currently supported.

There's currently no workaround for this issue.

### Unable to configure X.509 authentication for custom Kafka endpoints

---

Issue ID: 8750

---

Log signature: N/A

---

X.509 authentication for custom Kafka endpoints isn't currently supported.

### Data points aren't validated against a schema

---

Issue ID: 8794

---

Log signature: N/A

---

When you create a data flow, you can specify a schema in the source configuration. However, deserializing and validating messages using a schema isn't supported yet. Specifying a schema in the source configuration only allows the operations experience to display the list of data points, but the data points aren't validated against the schema.

### Connection failures with Azure Event Grid

---

Issue ID: 8891

---

Log signature: N/A

---

When you connect multiple IoT Operations instances to the same Event Grid MQTT namespace, connection failures might occur due to client ID conflicts. Client IDs are currently derived from data flow resource names, and when using infrastructure as code patterns for deployment, the generated client IDs might be identical.

To work around this issue, add randomness to the data flow names in your deployment templates.

### Data flow errors after a network disruption

---

Issue ID: 8953

---

Log signature: N/A

---

When the network connection is disrupted, data flows might encounter errors sending messages because of a mismatched producer ID.

To work around this issue, restart your data flow pods.

### Disconnections from Kafka endpoints

---

Issue ID: 9289

---

Log signature: N/A

---

If you use control characters in Kafka headers, you might encounter disconnections. Control characters in Kafka headers such as `0x01`, `0x02`, `0x03`, `0x04` are UTF-8 compliant but the IoT Operations MQTT broker rejects them. This issue happens during the data flow process when Kafka headers are converted to MQTT properties using a UTF-8 parser. Packets with control characters might be treated as invalid and rejected by the broker and lead to data flow failures.

To work around this issue, avoid using control characters in Kafka headers.

### Data flow deployment doesn't complete

---

Issue ID: 9411

---

Log signature:

`"Dataflow pod had error: Bad pod condition: Pod 'aio-dataflow-operator-0' container 'aio-dataflow-operator' stuck in a bad state due to 'CrashLoopBackOff'"`

`"Failed to create webhook cert resources: Failed to update ApiError: Internal error occurred: failed calling webhook "webhook.cert-manager.io" [...]"`

---

When you create a new data flow, it might not finish deployment. The cause is that the `cert-manager` wasn't ready or running.

To work around this issue, use the following steps to manually delete the data flow operator pod to clear the crash status:

1. Run `kubectl get pods -n azure-iot-operations`.
   In the output, Verify _aio-dataflow-operator-0_ is only data flow operator pod running.

1. Run `kubectl logs --namespace azure-iot-operations aio-dataflow-operator-0` to check the logs for the data flow operator pod.

   In the output, check for the final log entry:

   `Dataflow pod had error: Bad pod condition: Pod 'aio-dataflow-operator-0' container 'aio-dataflow-operator' stuck in a bad state due to 'CrashLoopBackOff'`

1. Run the _kubectl logs_ command again with the `--previous` option.

   `kubectl logs --namespace azure-iot-operations --previous aio-dataflow-operator-0`

   In the output, check for the final log entry:

   `Failed to create webhook cert resources: Failed to update ApiError: Internal error occurred: failed calling webhook "webhook.cert-manager.io" [...]`.

   If you see both log entries from the two _kubectl log_ commands, the cert-manager wasn't ready or running.

1. Run `kubectl delete pod aio-dataflow-operator-0 -n azure-iot-operations` to delete the data flow operator pod. Deleting the pod clears the crash status and restarts the pod.

1. Wait for the operator pod to restart and deploy the data flow.

### Data flows error metrics

---

Issue ID: 2382

---

Log signature: N/A

---

Data flows marks message retries and reconnects as errors, and as a result data flows might look unhealthy. This behavior is only seen in previous versions of data flows. Review the logs to determine if the data flow is healthy.
