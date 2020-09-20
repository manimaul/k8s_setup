# K8S Setup

This documents my Linode Kubernetes 1.17 cluster setup. We will start with a single Linode Cluster in Femont, CA and later expand to multiple clusters as needed. DNS will be provided by AWS Route53 so that we can do latency based load balancing on the multi-cluster.

Start with a 3 node cluster: 

|Plan|Monthly|Hourly|RAM|CPUs|Storage|
|----|-------|------|---|----|-------|
|2GB |$10    |$0.015|2GB|1   |50 GB  |

We will be installing the following components into the cluster:
* [Linkerd stable-2.8.1](https://linkerd.io/)
  * Service Mesh
* [HA Proxy](https://www.haproxy.com/documentation/kubernetes/latest/)
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

# Install HA Proxy

### install in cluster
I've edited [github.com/haproxytech/kubernetes-ingress/haproxy-ingress.yaml](https://raw.githubusercontent.com/haproxytech/kubernetes-ingress/v1.4.7/deploy/haproxy-ingress.yaml) to suit.
```
kubectl apply -f haproxy/haproxy_ingress.yaml
```

### inject linkerd
```
kubectl get -n haproxy-controller deploy -o yaml \
  | linkerd inject - \
  | kubectl apply -f -
```

### view dashboard
```
kubectl port-forward -n haproxy-controller haproxy-ingress-7d76995f66-8wjmz 8080:1024
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
curl https://raw.githubusercontent.com/manimaul/mxmariner.com/master/k8s.yml | linkerd inject - | kubectl apply -f -
```

# verify it works
```
echo | openssl s_client -showcerts -servername mxmariner.com -connect mxmariner.com:443 2>/dev/null | openssl x509 -inform pem -text | grep 'Issuer' 
```