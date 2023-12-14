# K8S Setup

This documents my Linode Kubernetes 1.21 cluster setup. DNS (provided by AWS Route53) points domain(s) to the NodeBalancer configured by the documented HA Proxy Ingress Controller. 

Start with a 3 node cluster: 

|Plan|Monthly|Hourly|RAM|CPUs|Storage|
|----|-------|------|---|----|-------|
|2GB |$10    |$0.015|2GB|1   |50 GB  |

We will be installing the following components into the cluster:
* [Istio](https://istio.io)
* [Cert Manager](https://cert-manager.io/)
  * x509 certificate management for Kubernetes

-------------------------------------------------

# Install Istio 

```shell
istioctl install
kubectl label namespace default istio-injection=enabled
kubectl label namespace mga istio-injection=enabled
kubectl apply -f ./istio_ingress_class.yaml
```
-------------------------------------------------

# Install Cert Manager

```shell
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.3/cert-manager.yaml
kubectl apply -f ./cluster-issuer.yaml
```

-------------------------------------------------

# Add a deployment 

```shell
kubectl create ns check
kubectl label namespace check istio-injection=enabled
kubectl apply -f ./check/k8s.yml
```

# verify it works

```shell
echo | openssl s_client -showcerts -servername check.manimaul.com -connect check.manimaul.com:443 2>/dev/null | openssl x509 -inform pem -text | grep 'Issuer' 
```

-------------------------------------------------

# Inject existing deployment
```shell
istioctl kube-inject -f <file>.yaml | kubectl apply -f -
```
