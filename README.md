# K8S Setup

This documents my Linode Kubernetes 1.17 cluster setup. We will start with a single Linode Cluster in Femont, CA and later expand to multiple clusters as needed. DNS will be provided by AWS Route53 so that we can do latency based load balancing on the multi-cluster.

Start with a 3 node cluster: 

|Plan|Monthly|Hourly|RAM|CPUs|Storage|
|----|-------|------|---|----|-------|
|2GB |$10    |$0.015|2GB|1   |50 GB  |

We will be installing the following components into the cluster:
* [Linkerd stable-2.8.1](https://linkerd.io/)
  * Service Mesh
* [Traefik 2.2.8](https://containo.us/traefik/)
  * Ingress Controller
* [Cert Manager](https://cert-manager.io/)
  * x509 certificate management for Kubernetes

-------------------------------------------------

# Linkerd Installation

### install locally 
`brew install linkerd`

### install in cluster 
`linkerd install | kubectl apply -f -`

### validate install
`linkerd check`

### see installed
`kubectl -n linkerd get deploy`

### view dashboard
`linkerd dashboard`

-------------------------------------------------

# Install Traefik 2

### install in cluster
```
helm repo add traefik https://containous.github.io/traefik-helm-chart
helm repo update
kubectl create namespace traefik
helm install --namespace traefik traefik traefik/traefik --values traefik/values.yaml
```

### inject linkerd
```
kubectl get -n traefik deploy -o yaml \
  | linkerd inject - \
  | kubectl apply -f -
```

### view dashboard
```
kubectl port-forward -n traefik $(kubectl get pods -n traefik --selector "app.kubernetes.io/name=traefik" --output=name) 9000:9000
```

-------------------------------------------------

# Install Cert Manager

### install in cluster
```
wget https://github.com/jetstack/cert-manager/releases/download/v1.0.1/cert-manager.yaml -o cert_manager/cert-manager.yaml

cat cert_manager/cert-manager.yaml | linkerd inject - | kubectl apply -f -

kubectl apply -f cert_manager/cluster-issuer.yaml
```

-------------------------------------------------

# Add a deployment 

```
cat mxmariner.com/mxmariner.yaml | linkerd inject - | kubectl apply -f -
```

# verify it works
```
echo | openssl s_client -showcerts -servername mxmariner.com -connect mxmariner.com:443 2>/dev/null | openssl x509 -inform pem -text | grep 'Issuer' 
```