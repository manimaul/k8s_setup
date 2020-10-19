# Install
```bash
curl -L https://github.com/jetstack/cert-manager/releases/download/v1.0.3/cert-manager.yaml > cert_manager/cert-manager.yaml
cat cert_manager/cert-manager.yaml | linkerd inject - | kubectl apply -f -
kubectl apply -f cert_manager/cluster-issuer.yaml
```
