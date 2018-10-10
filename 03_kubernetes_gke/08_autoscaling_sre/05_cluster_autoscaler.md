# Cluster Auto Scaler

In general, the cluster autoscaler is responsible for scaling up and down the underlying node pool, so that additional pods can be included and infrastructure remains utilized. In an ideal setup, you can focus on running the right number of pods, with the correct resources, and let the cluster autoscaler manage the size of your node pool accordingly.

The cluster auto scaler is generally implemented by the IaaS provider, for cloud deployments. There are autoscalers that connect to on-premise vm provisioning systems, to scale out that way as well.

Sorry, there is no minikube equivalent for this level.

## GKE

- Automatically resizes clusters based on workload demand
- Allows you to pay for only what you need at the moment (cost saving opportunity)
- Be aware that when resources are deleted or moved during an autoscale event, your services may experience disruption.  Ensure that your code accounts for this!
- **Caution**- Don't enable GCP Engine autoscaling for managed instance groups for your clusters nodes.  There will be issues as they are two seperate services.


Please refer to the official [GKE Cluster Autoscaler Documentation](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-autoscaler) for specifics.
