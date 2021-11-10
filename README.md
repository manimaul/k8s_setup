# K8S Setup

This documents my Linode Kubernetes 1.17 cluster setup. We will start with a single Linode Cluster in Femont, CA and later expand to multiple clusters as needed. DNS will be provided by AWS Route53 so that we can do latency based load balancing on the multi-cluster.

Start with a 3 node cluster: 

|Plan|Monthly|Hourly|RAM|CPUs|Storage|
|----|-------|------|---|----|-------|
|2GB |$10    |$0.015|2GB|1   |50 GB  |

We will be installing the following components into the cluster:
* [HA Proxy](https://www.haproxy.com/documentation/kubernetes/latest/)
  * Ingress Controller
* [Cert Manager](https://cert-manager.io/)
  * x509 certificate management for Kubernetes

-------------------------------------------------

# Install HA Proxy

### install in cluster
I've edited [github.com/haproxytech/kubernetes-ingress/haproxy-ingress.yaml](https://raw.githubusercontent.com/haproxytech/kubernetes-ingress/v1.7/deploy/haproxy-ingress.yaml) to suit.
```shell
kubectl apply -f haproxy/haproxy_ingress.yaml
```

### view dashboard
```shell

kubectl port-forward -n haproxy-controller $(kubectl get pods -n haproxy-controller -l run=haproxy-ingress -o jsonpath='{.items[*].metadata.name}') 8080:1024
# visit http://127.0.0.1:8080/
```

-------------------------------------------------

# Install Cert Manager

### install in cluster
```shell
wget https://github.com/jetstack/cert-manager/releases/download/v1.6.0/cert-manager.yaml 
kubectl apply -f cluster-issuer.yaml
kubectl apply -f cert-manager.yaml
```

-------------------------------------------------

# Add a deployment 

```shell
kubectl apply -f https://raw.githubusercontent.com/manimaul/mxmariner.com/master/k8s.yml
```

# verify it works

```shell
echo | openssl s_client -showcerts -servername mxmariner.com -connect mxmariner.com:443 2>/dev/null | openssl x509 -inform pem -text | grep 'Issuer' 
```