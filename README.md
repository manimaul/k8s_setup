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
  --set controller.service.type=LoadBalancer \
  --set controller.service.enablePorts.quic=false

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
  --version v1.16.3 \
  --set crds.enabled=true

kubectl apply -f ./cluster-issuer.yaml
```

# Metrics
https://github.com/kubernetes-sigs/metrics-server
```shell
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm upgrade --install metrics-server metrics-server/metrics-server
```

# Monitoring 

Install Prometheus + Grafana
```shell
kubectl create ns monitoring
mkdir monitor
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm search kube
helm show values prometheus-community/kube-prometheus-stack
export GRAFANA_ADMINPASSWORD="<a password>"
export HELM_CHART_VERSION="68.4.4"

#dry run 
helm install --namespace monitoring --dry-run \
  kube-prom-stack prometheus-community/kube-prometheus-stack \
  --version "${HELM_CHART_VERSION}" \
  --namespace monitoring \
  --values ./monitor/values.yaml \
  --set grafana.adminPassword=$GRAFANA_ADMINPASSWORD \
  --set prometheusOperator.createCustomResource=false > ./monitor/stack.yaml
  
#install
helm install --namespace monitoring \
  kube-prom-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --values ./monitor/values.yaml \
  --set grafana.adminPassword=$GRAFANA_ADMINPASSWORD \
  --set prometheusOperator.createCustomResource=false

#upgrade
helm upgrade --namespace monitoring \
  --values ./monitor/values.yaml \
  kube-prom-stack prometheus-community/kube-prometheus-stack \
  --version "${HELM_CHART_VERSION}" 
```

```shell
kubectl -n monitoring get svc

kubectl -n monitoring port-forward \
svc/kube-prom-stack-grafana 8080:80

kubectl -n monitoring port-forward \
svc/kube-prom-stack-kube-prome-prometheus 9090
```

Uninstall
```shell
helm uninstall kube-prom-stack
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
