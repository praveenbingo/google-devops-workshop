# Setup Kubernetes in Docker Desktop for macos

![step01](docker_desktop_macos_images/docker_desktop_for_macos-install-01.png)
![step02](docker_desktop_macos_images/docker_desktop_for_macos-install-02.png)
![step03](docker_desktop_macos_images/docker_desktop_for_macos-install-03.png)
![step04](docker_desktop_macos_images/docker_desktop_for_macos-install-04.png)
![step05](docker_desktop_macos_images/docker_desktop_for_macos-install-05.png)
![step06](docker_desktop_macos_images/docker_desktop_for_macos-install-06.png)
![step07](docker_desktop_macos_images/docker_desktop_for_macos-install-07.png)
![step08](docker_desktop_macos_images/docker_desktop_for_macos-install-08.png)

* Verify that `kubectl` can connect to the Kubernetes "server" running in Docker Desktop:
```
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"14", GitVersion:"v1.14.1", GitCommit:"b7394102d6ef778017f2ca4046abbaa23b88c290", GitTreeState:"clean", BuildDate:"2019-04-19T22:12:47Z", GoVersion:"go1.12.4", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"14", GitVersion:"v1.14.6", GitCommit:"96fac5cd13a5dc064f7d9f4f23030a6aeface6cc", GitTreeState:"clean", BuildDate:"2019-08-19T11:05:16Z", GoVersion:"go1.12.9", Compiler:"gc", Platform:"linux/amd64"}
```

* Check which context your kubeconfig is currently using/set to:
```
$ kubectl config current-context
docker-desktop
```

* Check all kubeconfig settings:
```
$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://kubernetes.docker.internal:6443
  name: docker-desktop
contexts:
- context:
    cluster: docker-desktop
    user: docker-desktop
  name: docker-desktop
- context:
    cluster: docker-desktop
    user: docker-desktop
  name: docker-for-desktop
current-context: docker-desktop
kind: Config
preferences: {}
users:
- name: docker-desktop
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

* Get information on the (single node) Kubernetes cluster running in Docker Desktop:
```
$ kubectl get nodes --output wide
NAME             STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE         KERNEL-VERSION     CONTAINER-RUNTIME
docker-desktop   Ready    master   13m   v1.14.6   192.168.65.3   <none>        Docker Desktop   4.9.184-linuxkit   docker://19.3.2
```

* Get a list of all Pods running in all Kubernetes namespaces:
```
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                     READY   STATUS    RESTARTS   AGE
docker        compose-6c67d745f6-crqbv                 1/1     Running   0          13m
docker        compose-api-57ff65b8c7-mxccg             1/1     Running   0          13m
kube-system   coredns-584795fc57-8xf49                 1/1     Running   0          14m
kube-system   coredns-584795fc57-rc8lh                 1/1     Running   0          14m
kube-system   etcd-docker-desktop                      1/1     Running   0          13m
kube-system   kube-apiserver-docker-desktop            1/1     Running   0          13m
kube-system   kube-controller-manager-docker-desktop   1/1     Running   0          13m
kube-system   kube-proxy-dsjth                         1/1     Running   0          14m
kube-system   kube-scheduler-docker-desktop            1/1     Running   0          13m
```
