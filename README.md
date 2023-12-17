# K8S Setup

This documents my Linode Kubernetes

We will be installing the following components into the cluster:
* [HA Proxy Ingress Controller](https://www.haproxy.com/documentation/kubernetes-ingress/community/installation/aws/)
* [Cert Manager](https://cert-manager.io/)

# Spin up Cluster 

- us-sea, 3 node, linode 4GB shared 2 cpu

# Install HA Proxy

```shell
helm repo add haproxytech https://haproxytech.github.io/helm-charts
helm repo update
helm install haproxy-kubernetes-ingress haproxytech/kubernetes-ingress \
  --create-namespace \
  --namespace haproxy-controller \
  --set controller.ingressClass=null \
  --set controller.service.type=LoadBalancer

kubectl -n haproxy-controller edit svc happroxy-kubernetes-ingress
# remove UDP port

# show loadbalancer public IP
kubectl -n haproxy-controller get svc 
```

# Install Cert Manager

```shell
helm repo add jetstack https://charts.jetstack.io --force-update

helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.16.2 \
  --set crds.enabled=true

kubectl apply -f ./cluster-issuer.yaml
```

-------------------------------------------------

# Add a deployment 

```shell
kubectl apply -f ./check.yaml
```

# verify it works

```shell
echo | openssl s_client -showcerts -servername check.manimaul.com -connect check.manimaul.com:443 2>/dev/null | openssl x509 -inform pem -text | grep 'Issuer' 
```
