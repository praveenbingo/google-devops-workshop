# Considerations for gRPC

gRPC is a modern open source high performance RPC framework that can run in any environment. It can efficiently connect services in and across data centers with pluggable support for load balancing, tracing, health checking and authentication. It is also applicable in last mile of distributed computing to connect devices, mobile applications and browsers to backend services.

However, gRPC also breaks the standard connection-level load balancing provided by Kubernetes. This is because gRPC is built on HTTP/2, and HTTP/2 is designed to have a single long-lived TCP connection, across which all requests are multiplexed.

In network terms, this means we need to make decisions at L5/L7 rather than L3/L4, i.e. we need to understand the protocol sent over the TCP connections.

How do we accomplish this? There are a couple options. First, our application code could manually maintain its own load balancing “pool” of destinations, and we could configure our gRPC client to use this load balancing pool. This approach gives us the most control, but it can be very complex in environments like Kubernetes where the pool changes over time as Kubernetes reschedules pods.

Our application would have to watch the Kubernetes API and keep itself up to date with the pods. Alternatively, in Kubernetes, we could deploy our app as headless services. In this case, Kubernetes will create multiple A records in the DNS entry for the service. If our gRPC client is sufficiently advanced, it can automatically maintain the load balancing pool from those DNS entries. But this approach restricts us to certain gRPC clients, and it’s rarely possible to only use headless services.

Finally, we can take a third approach: use a lightweight proxy.

## Introduce a Proxy / Service Mesh

### gRPC with nginx Ingress

https://github.com/kubernetes/ingress-nginx/tree/master/docs/examples/grpc


### gRPC with Istio Gateway

https://cloud.google.com/solutions/using-istio-for-internal-load-balancing-of-grpc-services

1. Create a Kubernetes Cluster

`gcloud auth login`

`gcloud config set project PROJECT_ID`

```
gcloud services enable \
    cloudapis.googleapis.com \
    cloudbuild.googleapis.com \
    container.googleapis.com \
    containeranalysis.googleapis.com
```

`gcloud config set compute/zone us-west1-a`

```
cd grpc-greeter-go
```

Create a new GKE Cluster
```
gcloud beta container clusters create kubernetes-workshop \
    --machine-type n1-standard-2 \
    --enable-ip-alias
```

or Connect to an existing Cluster
```
gcloud container clusters get-credentials kubernetes-workshop --zone us-west1-a --project PROJECT_ID
```

2. Add role binding, for your user.

```
kubectl create clusterrolebinding cluster-admin-binding \
    --clusterrole cluster-admin \
    --user $(gcloud config get-value account)
```

3. Create istio namespace

`kubectl create namespace istio-system`

3. Download Istio & Helm Cli Tools

```
export ISTIO_VERSION=1.1.10
curl -L https://git.io/getLatestIstio | sh -
cd istio-1.1.10
export PATH=$(pwd)/bin:$PATH
istioctl version
```

```
curl -LO https://git.io/get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
helm version
```

4. Install Istio onto the Cluster

`cd istio-1.1.10`

```
helm template \
    install/kubernetes/helm/istio-init \
    --name istio-init \
    --namespace istio-system | kubectl apply -f -
```

```
export ISTIO_VERSION=1.1.10
export ISTIO_PACKAGE=1.1.10-gke.0

helm template \
    install/kubernetes/helm/istio \
    --set gateways.istio-ingressgateway.enabled=false \
    --set gateways.istio-ilbgateway.enabled=true \
    --set gateways.istio-ilbgateway.ports[0].name=grpc \
    --set gateways.istio-ilbgateway.ports[0].port=443 \
    --set global.hub=gcr.io/gke-release/istio \
    --set global.tag=$ISTIO_PACKAGE \
    --set prometheus.hub=gcr.io/gke-release/istio/prom \
    --name istio \
    --namespace istio-system | kubectl apply -f -
```

`kubectl get services istio-ilbgateway -n istio-system --watch`

5. Create TLS Certificates

```
openssl req -x509 -nodes -newkey rsa:2048 -days 365 \
    -keyout privkey.pem -out cert.pem -subj "/CN=grpc.example.com"
```

```
kubectl -n istio-system create secret tls istio-ilbgateway-certs \
    --key privkey.pem --cert cert.pem \
    --dry-run -o yaml | kubectl apply -f -
```

6. Create a namespace for your application, could be default.

`kubectl create namespace testing`

No matter what you called your namespace, the following should be applied to the chosen namespace.

`kubectl label namespace testing istio-injection=enabled`

7. Build the GRPC Sample App Server Docker Image

`gcloud builds submit server -t gcr.io/$GOOGLE_CLOUD_PROJECT/grpc-greeter-go-server`

or build locally for docker desktop (you will need to specify `imagePullPolicy: Never` in the later step.)

`docker build server -t grpc-greeter-go-server:latest`

or you can use the pre-built image on redaptcloud dockerhub

`redaptcloud/grpc-greeter-go-server`

8. Deploy the sample app server

In the `manifests` folder of `grpc-greeter-go` you will see `greeter-k8s.template.yaml`, we will need to update this deployment to reflect the image built in the previous step.

`cp manifests/greeter-k8s.template.yaml manifests/greeter-k8s.yaml`

In the copy without the extension, modify this line: `image: gcr.io/$GOOGLE_CLOUD_PROJECT/grpc-greeter-go-server`

If you are testing on Docker Desktop Kubernetes, you will need to change to `imagePullPolicy: Never`

Note: Be sure to choose the namespace specified above.

`kubectl apply --namespace testing -f manifests/greeter-k8s.yaml`

`kubectl get services,pods -n testing`

9. Deploy the Istio Objects

```
kubectl apply --namespace testing -f manifests/greeter-istio-ilbgateway.yaml \
    -f manifests/greeter-istio-virtualservice.yaml \
    -f manifests/greeter-istio-destinationrule.yaml
```

`kubectl get gateway,virtualservice,destinationrule -n testing`

10.  Build the GRPC Sample App Client Docker Image

`gcloud builds submit client -t gcr.io/$GOOGLE_CLOUD_PROJECT/grpc-greeter-go-client`

or build locally for docker desktop (you will need to specify `imagePullPolicy: Never` in the later step.)

`docker build client -t grpc-greeter-go-client:latest`

or you can use the pre-built image on redaptcloud dockerhub

`redaptcloud/grpc-greeter-go-client`

11. Verify that gRPC Load Balancing is taking place, by launching our client in the istio-system namespace and referencing its secret for tls.

`kubectl apply -n istio-system -f manifests/greeter-client-test.yaml`

Note: You can deploy this to another namespace, but the TLS secret above will need to be copied to that namespace.

`kubectl get pods -n istio-system`

`kubectl logs -f -n istio-system greeter-client-XXXXXX`

Your output should show something like:

```
2019/10/09 14:21:32 Hello grpc.example.com:443 from greeter-6cbdf6bcf7-7wqg6
2019/10/09 14:21:32 Hello grpc.example.com:443 from greeter-6cbdf6bcf7-7z248
2019/10/09 14:21:32 Hello grpc.example.com:443 from greeter-6cbdf6bcf7-dh5fl
2019/10/09 14:21:32 Hello grpc.example.com:443 from greeter-6cbdf6bcf7-7wqg6
2019/10/09 14:21:32 Hello grpc.example.com:443 from greeter-6cbdf6bcf7-7z248
2019/10/09 14:21:32 Hello grpc.example.com:443 from greeter-6cbdf6bcf7-dh5fl
2019/10/09 14:21:32 Hello grpc.example.com:443 from greeter-6cbdf6bcf7-7wqg6
2019/10/09 14:21:32 Hello grpc.example.com:443 from greeter-6cbdf6bcf7-7z248
2019/10/09 14:21:32 Hello grpc.example.com:443 from greeter-6cbdf6bcf7-dh5fl
```


### gRPC with LinkerD

https://linkerd.io/2018/11/14/grpc-load-balancing-on-kubernetes-without-tears/

### gRPC with Google Endpoints

https://cloud.google.com/endpoints/docs/grpc/get-started-kubernetes-engine