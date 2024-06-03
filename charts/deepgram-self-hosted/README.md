# deepgram-self-hosted

![Version: 0.1.1-alpha](https://img.shields.io/badge/Version-0.1.1--alpha-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: release-240528](https://img.shields.io/badge/AppVersion-release--240528-informational?style=flat-square) [![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/deepgram-self-hosted)](https://artifacthub.io/packages/search?repo=deepgram-self-hosted)

A Helm chart for running Deepgram services in a self-hosted environment

**Homepage:** <https://developers.deepgram.com/docs/self-hosted-introduction>

**Deepgram Self-Hosted Kubernetes Guides:** <https://developers.deepgram.com/docs/kubernetes>

## Source Code

* <https://github.com/deepgram/self-hosted-resources>

## Requirements

Kubernetes: `>=1.27.0-0`

| Repository | Name | Version |
|------------|------|---------|
| https://helm.ngc.nvidia.com/nvidia | gpu-operator | ^24.3.0 |

## Get Repository Info

```bash
helm repo add deepgram https://deepgram.github.io/self-hosted-resources
helm repo update
```

## Installing the Chart

The Deepgram self-hosted chart requires Helm 3.7+ in order to install successfully. Please check your helm release before installation.

You will need to provide your [self-service Deepgram licensing and credentials](https://developers.deepgram.com/docs/on-prem-self-service-tutorial) information. See `global.deepgramSecretRef` and `global.pullSecretRef` in the [Values section](#values) for more details.

You may also override any default configuration values. See [the Values section](#values) for a list of available options, and the [samples directory](./samples) for examples of a standard installation.

```
helm install -f my-values.yaml [RELEASE_NAME] deepgram/deepgram-self-hosted --atomic --timeout 20m
```

## Upgrade and Rollback Strategies

To upgrade the Deepgram components to a new version, follow these steps:

1. Update the various `image.tag` values in the `values.yaml` file to the desired version.

2. Run the Helm upgrade command:

    ```bash
    helm upgrade -f my-values.yaml [RELEASE_NAME] deepgram/deepgram-self-hosted --atomic --timeout 30m
    ```

If you encounter any issues during the upgrade process, you can perform a rollback to the previous version:

```bash
helm rollback deepgram
```

Before upgrading, ensure that you have reviewed the release notes and any migration guides provided by Deepgram for the specific version you are upgrading to.

## Uninstalling the Chart

```bash
helm uninstall [RELEASE_NAME]
```

This removes all the Kubernetes components associated with the chart and deletes the release.

## Persistent Storage Options

The Deepgram Helm chart supports different persistent storage options for storing Deepgram models and data. The available options include:

- AWS Elastic File System (EFS)
- Google Cloud Persistent Disk (GPD)
- Custom PersistentVolumeClaim (PVC)

To configure a specific storage option, see the `engine.modelManager.volumes` [configuration values](#values). Make sure to provide the necessary configuration values for the selected storage option, such as the EFS file system ID or the GPD disk type and size.

For detailed instructions on setting up and configuring each storage option, refer to the [Deepgram self-hosted guides](https://developers.deepgram.com/docs/kubernetes) and the respective cloud provider's documentation.

## RBAC Configuration

Role-Based Access Control (RBAC) is used to control access to Kubernetes resources based on the roles and permissions assigned to users or service accounts. The Deepgram Helm chart includes default RBAC roles and bindings for the API, Engine, and License Proxy components.

To use custom RBAC roles and bindings based on your specific security requirements, you can individually specify pre-existing ServiceAccounts to bind to each deployment by specifying the following options in `values.yaml`:

```
{api|engine|licenseProxy}.serviceAccount.create=false
{api|engine|licenseProxy}.serviceAccount.name=<your-pre-existing-sa>
```

Make sure to review and adjust the RBAC configuration according to the principle of least privilege, granting only the necessary permissions for each component.

## Secret Management

The Deepgram Helm chart takes references to two existing secrets - one containing your distribution credentials to pull container images from Deepgram's image repository, and one containing your Deepgram self-hosted API key.

Consult the [official Kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/secret/) for best practices on configuring Secrets for use in your cluster.

## Getting Help

See the [Getting Help](../../README.md#getting-help) section in the root of this repository for a list of resources to help you troubleshoot and resolve issues.

### Troubleshooting

If you encounter issues while deploying or using Deepgram, consider the following troubleshooting steps:

1. Check the pod status and logs:
   - Use `kubectl get pods` to check the status of the Deepgram pods.
   - Use `kubectl logs <pod-name>` to view the logs of a specific pod.

2. Verify resource availability:
   - Ensure that the cluster has sufficient CPU, memory, and storage resources to accommodate the Deepgram components.
   - Check for any resource constraints or limits imposed by the namespace or the cluster.

3. Review the Kubernetes events:
   - Use `kubectl get events` to view any events or errors related to the Deepgram deployment.

4. Check the network connectivity:
   - Verify that the Deepgram components can communicate with each other and with the Deepgram license server (license.deepgram.com).
   - Check the network policies and firewall rules to ensure that the necessary ports and protocols are allowed.

5. Collect diagnostic information:
   - Gather relevant logs and metrics.
   - Export your existing Helm chart values:
       ```bash
       helm get values [RELEASE_NAME] > my-deployed-values.yaml
       ```
   - Provide the collected diagnostic information to Deepgram for assistance.

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| api.additionalLabels | object | `{}` | Additional labels to add to API resources |
| api.affinity | object | `{}` | [Affinity and anti-affinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity) to apply for API pods. |
| api.concurrencyLimit.activeRequests | string | `nil` | activeRequests limits the number of active requests handled by a single API container. If additional requests beyond the limit are sent, API will return a 429 HTTP status code. Set to an integer if desired, otherwise the `nil` default means no limit will be set. |
| api.driverPool | object | `` | driverPool configures the backend pool of speech engines (generically referred to as "drivers" here). The API will load-balance among drivers in the standard pool; if one standard driver fails, the next one will be tried. |
| api.driverPool.standard | object | `` | standard is the main driver pool to use. |
| api.driverPool.standard.maxResponseSize | string | `"1073741824"` | Maximum response to deserialize from Driver (in bytes). Default is 1GB, expressed in bytes. |
| api.driverPool.standard.retryBackoff | float | `1.6` | retryBackoff is the factor to increase the retrySleep by for each additional retry (for exponential backoff). |
| api.driverPool.standard.retrySleep | string | `"2s"` | retrySleep defines the initial sleep period (in humantime duration) before attempting a retry. |
| api.driverPool.standard.timeoutBackoff | float | `1.2` | timeoutBackoff is the factor to increase the timeout by for each additional retry (for exponential backoff). |
| api.features | object | `` | Enable ancillary features |
| api.features.diskBufferPath | string | `nil` | If API is receiving requests faster than Engine can process them, a request queue will form. By default, this queue is stored in memory. Under high load, the queue may grow too large and cause Out-Of-Memory errors. To avoid this, set a diskBufferPath to buffer the overflow on the request queue to disk.  WARN: This is only to temporarily buffer requests during high load. If there is not enough Engine capacity to process the queued requests over time, the queue (and response time) will grow indefinitely. |
| api.features.summarization | bool | `true` | summarization enable summarization *if* a valid summarization model is available. |
| api.features.topicDetection | bool | `true` | topicDetection enables topic detection *if* a valid topic detection model is available. |
| api.image.path | string | `"quay.io/deepgram/onprem-api"` | path configures the image path to use for creating API containers. You may change this from the public Quay image path if you have imported Deepgram images into a private container registry. |
| api.image.pullPolicy | string | `"IfNotPresent"` | pullPolicy configures how the Kubelet attempts to pull the Deepgram API image |
| api.image.tag | string | `"release-240528"` | tag defines which Deepgram release to use for API containers |
| api.livenessProbe | object | `` | Liveness probe customization for API pods. |
| api.namePrefix | string | `"deepgram-api"` | namePrefix is the prefix to apply to the name of all K8s objects associated with the Deepgram API containers. |
| api.readinessProbe | object | `` | Readiness probe customization for API pods. |
| api.resolver | object | `` | Specify custom DNS resolution options. |
| api.resolver.maxTTL | int | `nil` | maxTTL sets the DNS TTL value if specifying a custom DNS nameserver. |
| api.resolver.nameservers | list | `[]` | nameservers allows for specifying custom domain name server(s). A valid list item's format is "{IP} {PORT} {PROTOCOL (tcp or udp)}", e.g. `"127.0.0.1 53 udp"`. |
| api.resources | object | `` | Configure resource limits per API container. See [Deepgram's documentation](https://developers.deepgram.com/docs/on-prem-deployment-environments#api) for more details. |
| api.securityContext | object | `{}` | [Security context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/) for API pods. |
| api.server | object | `` | Configure how the API will listen for your requests |
| api.server.callbackConnTimeout | string | `"1s"` | callbackConnTimeout configures how long to wait for a connection to a callback URL. See [Deepgram's callback documentation](https://developers.deepgram.com/docs/callback) for more details. The value should be a humantime duration. |
| api.server.callbackTimeout | string | `"10s"` | callbackTimeout configures how long to wait for a response from a callback URL. See [Deepgram's callback documentation](https://developers.deepgram.com/docs/callback) for more details. The value should be a humantime duration. |
| api.server.fetchConnTimeout | string | `"1s"` | fetchConnTimeout configures how long to wait for a connection to a fetch URL. The value should be a humantime duration. A fetch URL is a URL passed in an inference request from which a payload should be downloaded. |
| api.server.fetchTimeout | string | `"60s"` | fetchTimeout configures how long to wait for a response from a fetch URL. The value should be a humantime duration. A fetch URL is a URL passed in an inference request from which a payload should be downloaded. |
| api.server.host | string | `"0.0.0.0"` | host is the IP address to listen on. You will want to listen on all interfaces to interact with other pods in the cluster. |
| api.server.port | int | `8080` | port to listen on. |
| api.serviceAccount.create | bool | `true` | Specifies whether to create a default service account for the Deepgram API Deployment. |
| api.serviceAccount.name | string | `nil` | Allows providing a custom service account name for the API component. If left empty, the default service account name will be used. If specified, and `api.serviceAccount.create = true`, this defines the name of the default service account. If specified, and `api.serviceAccount.create = false`, this provides the name of a preconfigured service account you wish to attach to the API deployment. |
| api.tolerations | list | `[]` | [Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) to apply to API pods. |
| api.updateStrategy.rollingUpdate.maxSurge | int | `1` | The maximum number of extra API pods that can be created during a rollingUpdate, relative to the number of replicas. See the [Kubernetes documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-surge) for more details. |
| api.updateStrategy.rollingUpdate.maxUnavailable | int | `0` | The maximum number of API pods, relative to the number of replicas, that can go offline during a rolling update. See the [Kubernetes documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-unavailable) for more details. |
| engine.additionalLabels | object | `{}` | Additional labels to add to Engine resources |
| engine.affinity | object | `{}` | [Affinity and anti-affinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity) to apply for Engine pods. |
| engine.chunking | object | `` | chunking defines the size of STT audio chunks to process in seconds. Adjusting these values will affect both inference performance and accuracy of results. Please contact your Deepgram Account Representative if you want to adjust any of these values. |
| engine.chunking.batch.maxDuration | string | `nil` | minDuration is the maximum audio duration for a STT chunk size for a batch request |
| engine.chunking.batch.minDuration | string | `nil` | minDuration is the minimum audio duration for a STT chunk size for a batch request |
| engine.chunking.streaming.maxDuration | string | `nil` | minDuration is the maximum audio duration for a STT chunk size for a streaming request |
| engine.chunking.streaming.minDuration | string | `nil` | minDuration is the minimum audio duration for a STT chunk size for a streaming request |
| engine.chunking.streaming.step | float | `1` | step defines how often to return interim results, in seconds. This value may be lowered to increase the frequency of interim results. However, this also causes a significant decrease in the number of concurrent streams supported by a single GPU. Please contact your Deepgram Account representative for more details. |
| engine.features | object | `` | Enable ancillary features |
| engine.features.languageDetection | bool | `true` | languageDetection enables Deepgram language detection *if* a valid language detection model is available |
| engine.features.multichannel | bool | `true` | multichannel allows/disallows multichannel requests |
| engine.halfPrecision.state | string | `"auto"` | Engine will automatically enable half precision operations if your GPU supports them. You can explicitly enable or disable this behavior with the state parameter which supports `"enable"`, `"disabled"`, and `"auto"`. |
| engine.image.path | string | `"quay.io/deepgram/onprem-engine"` | path configures the image path to use for creating Engine containers. You may change this from the public Quay image path if you have imported Deepgram images into a private container registry. |
| engine.image.pullPolicy | string | `"IfNotPresent"` | pullPolicy configures how the Kubelet attempts to pull the Deepgram Engine image |
| engine.image.tag | string | `"release-240528"` | tag defines which Deepgram release to use for Engine containers |
| engine.livenessProbe | object | `` | Liveness probe customization for Engine pods. |
| engine.metricsServer | object | `` | metricsServer exposes an endpoint on each Engine container for reporting inference-specific system metrics. See https://developers.deepgram.com/docs/metrics-guide#deepgram-engine for more details. |
| engine.metricsServer.host | string | `"0.0.0.0"` | host is the IP address to listen on for metrics requests. You will want to listen on all interfaces to interact with other pods in the cluster. |
| engine.metricsServer.port | int | `9991` | port to listen on for metrics requests |
| engine.modelManager.models.links | list | `[]` | Links to your Deepgram models, if automatically downloading into storage backing a persistent volume. **Automatic downloads are currently supported for AWS EFS volumes only.** Insert each model link provided to you by your Deepgram Account Representative. |
| engine.modelManager.searchPaths | list | `["/models"]` | searchPaths enumerates directories where inference models may be stored. |
| engine.modelManager.volumes.aws.efs.enabled | bool | `false` | Whether to use an [AWS Elastic File Sytem](https://aws.amazon.com/efs/) to store Deepgram models for use by Engine containers. This option requires your cluster to be running in [AWS EKS](https://aws.amazon.com/eks/). |
| engine.modelManager.volumes.aws.efs.fileSystemId | string | `nil` | FileSystemId of existing AWS Elastic File System where Deepgram model files will be persisted. You can find it using the AWS CLI: ``` $ aws efs describe-file-systems --query "FileSystems[*].FileSystemId" ``` |
| engine.modelManager.volumes.aws.efs.forceDownload | bool | `false` | Whether to force a fresh download of all model links provided, even if models are already present in EFS. |
| engine.modelManager.volumes.aws.efs.namePrefix | string | `"dg-models"` | Name prefix for the resources associated with the model storage in AWS EFS. |
| engine.modelManager.volumes.customVolumeClaim.enabled | bool | `false` | You may manually create your own PersistentVolume and PersistentVolumeClaim to store and expose model files to the Deepgram Engine. Configure your storage beforehand, and enable here. Note: Make sure the PV and PVC accessMode are set to `readWriteMany` or `readOnlyMany` |
| engine.modelManager.volumes.customVolumeClaim.name | string | `nil` | Name of your pre-configured PersistentVolumeClaim |
| engine.modelManager.volumes.gcp.gpd.enabled | bool | `false` | Whether to use an [GKE Persistent Disks](https://cloud.google.com/kubernetes-engine/docs/concepts/persistent-volumes) to store Deepgram models for use by Engine containers. This option requires your cluster to be running in [GCP GKE](https://cloud.google.com/kubernetes-engine). See the GKE documentation on [using pre-existing persistent disks](https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/preexisting-pd). |
| engine.modelManager.volumes.gcp.gpd.fsType | string | `"ext4"` |  |
| engine.modelManager.volumes.gcp.gpd.namePrefix | string | `"dg-models"` | Name prefix for the resources associated with the model storage in GCP GPD. |
| engine.modelManager.volumes.gcp.gpd.storageCapacity | string | `"40G"` | The size of your pre-existing persistent disk. |
| engine.modelManager.volumes.gcp.gpd.storageClassName | string | `"standard-rwo"` | The storageClassName of the existing persistent disk. |
| engine.modelManager.volumes.gcp.gpd.volumeHandle | string | `""` | The identifier of your pre-existing persistent disk. The format is projects/{project_id}/zones/{zone_name}/disks/{disk_name} for Zonal persistent disks, or projects/{project_id}/regions/{region_name}/disks/{disk_name} for Regional persistent disks. |
| engine.namePrefix | string | `"deepgram-engine"` | namePrefix is the prefix to apply to the name of all K8s objects associated with the Deepgram Engine containers. |
| engine.readinessProbe | object | `` | Readiness probe customization for Engine pods. |
| engine.resources | object | `` | Configure resource limits per Engine container. See [Deepgram's documentation](https://developers.deepgram.com/docs/on-prem-deployment-environments#engine) for more details. |
| engine.resources.limits.gpu | int | `1` | gpu maps to the nvidia.com/gpu resource parameter |
| engine.resources.requests.gpu | int | `1` | gpu maps to the nvidia.com/gpu resource parameter |
| engine.securityContext | object | `{}` | [Security context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/) for API pods. |
| engine.server | object | `` | Configure Engine containers to listen for requests from API containers. |
| engine.server.host | string | `"0.0.0.0"` | host is the IP address to listen on for inference requests. You will want to listen on all interfaces to interact with other pods in the cluster. |
| engine.server.port | int | `8080` | port to listen on for inference requests |
| engine.serviceAccount.create | bool | `true` | Specifies whether to create a default service account for the Deepgram Engine Deployment. |
| engine.serviceAccount.name | string | `nil` | Allows providing a custom service account name for the Engine component. If left empty, the default service account name will be used. If specified, and `engine.serviceAccount.create = true`, this defines the name of the default service account. If specified, and `engine.serviceAccount.create = false`, this provides the name of a preconfigured service account you wish to attach to the Engine deployment. |
| engine.startupProbe | object | `` | The startupProbe combination of `periodSeconds` and `failureThreshold` allows time for the container to load all models and start listening for incoming requests.  Model load time can be affected by hardware I/O speeds, as well as network speeds if you are using a network volume mount for the models.  If you are hitting the failure threshold before models are finished loading, you may want to extend the startup probe. However, this will also extend the time it takes to detect a pod that can't establish a network connection to validate its license. |
| engine.startupProbe.failureThreshold | int | `60` | failureThreshold defines how many unsuccessful startup probe attempts are allowed before the container will be marked as Failed |
| engine.tolerations | list | `[]` | [Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) to apply to Engine pods. |
| engine.updateStrategy.rollingUpdate.maxSurge | int | `1` | The maximum number of extra Engine pods that can be created during a rollingUpdate, relative to the number of replicas. See the [Kubernetes documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-surge) for more details. |
| engine.updateStrategy.rollingUpdate.maxUnavailable | int | `0` | The maximum number of Engine pods, relative to the number of replicas, that can go offline during a rolling update. See the [Kubernetes documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-unavailable) for more details. |
| global.additionalLabels | object | `{}` | Additional labels to add to all Deepgram resources |
| global.deepgramSecretRef | string | `nil` | deepgramSecretRef is the name of the pre-configured K8s Secret containing your Deepgram self-hosted API key. See chart docs for more details. |
| global.outstandingRequestGracePeriod | int | `1800` | When an API or Engine container is signaled to shutdown via Kubernetes sending a SIGTERM signal, the container will stop listening on its port, and no new requests will be routed to that container. However, the container will continue to run until all existing batch or streaming requests have completed, after which it will gracefully shut down.  Batch requests should be finished within 10-15 minutes, but streaming requests can proceed indefinitely.  outstandingRequestGracePeriod defines the period (in sec) after which Kubernetes will forcefully shutdown the container, terminating any outstanding connections. |
| global.pullSecretRef | string | `nil` | pullSecretRef is the name of the pre-configured K8s Secret with image repository credentials. See chart docs for more details. |
| gpu-operator | object | `{"driver":{"enabled":true,"version":"550.54.15"},"enabled":true,"toolkit":{"enabled":true,"version":"v1.15.0-ubi8"}}` | Passthrough values for [NVIDIA GPU Operator Helm chart](https://github.com/NVIDIA/gpu-operator/blob/master/deployments/gpu-operator/values.yaml) You may use the NVIDIA GPU Operator to manage installation of NVIDIA drivers and the container toolkit on nodes with attached GPUs. |
| gpu-operator.driver.enabled | bool | `true` | Whether to install NVIDIA drivers on nodes where a NVIDIA GPU is detected. If your Kubernetes nodes run a base image that comes with NVIDIA drivers pre-configured, disable this option, but keep the parent `gpu-operator` and sibling `toolkit` options enabled. |
| gpu-operator.driver.version | string | `"550.54.15"` | NVIDIA driver version to install. |
| gpu-operator.enabled | bool | `true` | Whether to install the NVIDIA GPU Operator to manage driver and/or container toolkit installation. See the list of [supported Operating Systems](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/platform-support.html#supported-operating-systems-and-kubernetes-platforms) to verify compatibility with your cluster/nodes. Disable this option if your cluster/nodes are not compatible. If disabled, you will need to self-manage NVIDIA software installation on all nodes where you want to schedule Deepgram Engine pods. |
| gpu-operator.toolkit.enabled | bool | `true` | Whether to install NVIDIA drivers on nodes where a NVIDIA GPU is detected. |
| gpu-operator.toolkit.version | string | `"v1.15.0-ubi8"` | NVIDIA container toolkit to install. The default `ubuntu` image tag for the toolkit requires a dynamic runtime link to a version of GLIBC that may not be present on nodes running older Linux distribution releases, such as Ubuntu 22.04. Therefore, we specify the `ubi8` image, which statically links the GLIBC library and avoids this issue. |
| licenseProxy | object | `` | Configuration options for the optional [Deepgram License Proxy](https://developers.deepgram.com/docs/license-proxy). |
| licenseProxy.additionalLabels | object | `{}` | Additional labels to add to License Proxy resources |
| licenseProxy.affinity | object | `{}` | [Affinity and anti-affinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity) to apply for License Proxy pods. |
| licenseProxy.image.path | string | `"quay.io/deepgram/onprem-license-proxy"` | path configures the image path to use for creating License Proxy containers. You may change this from the public Quay image path if you have imported Deepgram images into a private container registry. |
| licenseProxy.image.pullPolicy | string | `"IfNotPresent"` | pullPolicy configures how the Kubelet attempts to pull the Deepgram License Proxy image |
| licenseProxy.image.tag | string | `"release-240528"` | tag defines which Deepgram release to use for License Proxy containers |
| licenseProxy.livenessProbe | object | `` | Liveness probe customization for Proxy pods. |
| licenseProxy.namePrefix | string | `"deepgram-license-proxy"` | namePrefix is the prefix to apply to the name of all K8s objects associated with the Deepgram License Proxy containers. |
| licenseProxy.readinessProbe | object | `` | Readiness probe customization for License Proxy pods. |
| licenseProxy.resources | object | `` | Configure resource limits per License Proxy container. See [Deepgram's documentation](https://developers.deepgram.com/docs/license-proxy#system-requirements) for more details. |
| licenseProxy.securityContext | object | `{}` | [Security context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/) for API pods. |
| licenseProxy.server | object | `` | Configure how the license proxy will listen for licensing requests. |
| licenseProxy.server.baseUrl | string | `"/"` | baseUrl is the prefix for incoming license verification requests. |
| licenseProxy.server.host | string | `"0.0.0.0"` | host is the IP address to listen on. You will want to listen on all interfaces to interact with other pods in the cluster. |
| licenseProxy.server.port | int | `8443` | port to listen on. |
| licenseProxy.server.statusPort | int | `8080` | statusPort is the port to listen on for the status/health endpoint. |
| licenseProxy.serviceAccount.create | bool | `true` | Specifies whether to create a default service account for the Deepgram License Proxy Deployment. |
| licenseProxy.serviceAccount.name | string | `nil` | Allows providing a custom service account name for the LicenseProxy component. If left empty, the default service account name will be used. If specified, and `licenseProxy.serviceAccount.create = true`, this defines the name of the default service account. If specified, and `licenseProxy.serviceAccount.create = false`, this provides the name of a preconfigured service account you wish to attach to the License Proxy deployment. |
| licenseProxy.tolerations | list | `[]` | [Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) to apply to License Proxy pods. |
| licenseProxy.updateStrategy.rollingUpdate | object | `` | For the LicenseProxy, we only expose maxSurge and not maxUnavailable. This is to avoid accidentally having all LicenseProxy nodes go offline during upgrades, which could impact the entire cluster's connection to the Deepgram License Server. |
| licenseProxy.updateStrategy.rollingUpdate.maxSurge | int | `1` | The maximum number of extra License Proxy pods that can be created during a rollingUpdate, relative to the number of replicas. See the [Kubernetes documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-surge) for more details. |
| scaling | object | `` | Configuration options for horizontal scaling of Deepgram services. |
| scaling.static.api.replicas | int | `1` | Number of API pods to deploy. |
| scaling.static.engine.replicas | int | `1` | Number of Engine pods to deploy. |
| scaling.static.licenseProxy | object | `` | The License Proxy is optional, but highly recommended to be deployed in production to enable highly available environments. If deployed, one replica should be sufficient to support many API/Engine pods. |
| scaling.static.licenseProxy.replicas | int | `0` | Number of License Proxy pods to deploy. |

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| Deepgram Self-Hosted | <self.hosted@deepgram.com> |  |