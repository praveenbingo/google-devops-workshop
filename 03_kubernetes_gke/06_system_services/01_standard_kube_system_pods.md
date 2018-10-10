# Standard Kube System Pods

When you run the command, `kubectl get pods -n kube-system`, you are viewing the pods in the `kube-system` namespace. This namespace is typically reserved for cluster level services, such as autoscalar, external dns, kube proxy, heapster, kube dns, kubernetes dashboard, etc.

## Standard

The standard cluster services that are generally available on all clusters, are `kube-proxy`, `kube-dns`.

In unmanaged installations, its common to find these components running as containers, on the cluster as well: `etcd`, `kube-apiserver`, `dns-controller`, `kube-controller-manager`, `kube-scheduler`

These services correspond to the components we described earlier in this section.

## GKE

In GKE, there are special services deployed to your cluster such as `fluentd`, `event-exporter`, `heapster`, `kubernetes-dashboard`, `l7-default-backend`, and `metrics-server`

1. **fluentd** - Logging agent that works with stackdiver logging.  Used to collect logs from containers.
2. **event-exporter** - Kubernetes events are objects that provide insight into what is happening inside a cluster, such as what decisions were made by scheduler or why some pods were evicted from the node. 
3. **heapster** [**_Deprecated -- use metrics-server instead_**] - Heapster enables Container Cluster Monitoring and Performance Analysis for Kubernetes (versions v1.0.6 and higher), and platforms which include it. 
4. **kubernetes-dashboard**- Legacy K8s Dashboard (optional)
5. **l7-default-backend** - Ingress.  Provides layer 7 Load balancing.
6. **metics-server** - Cluster-wide resource usage aggregator. Collects metrics from the Summary API, exposed by Kubelet on each node.