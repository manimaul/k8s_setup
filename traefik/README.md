

  
-----------------OLD down

```
helm install --dry-run traefik traefik/traefik > traefik/traefik.yml
<edit traefik.yml>
cat ./traefik/traefik.yml | linkerd inject - | kubectl apply -f -
```

* view logs
```
kubectl -n traefikingress get pods
kubectl -n traefik logs $(kubectl -n traefik get pods --selector "app.kubernetes.io/name=traefik" --output=name) -f
```

* port forward into dashboard
```
kubectl -n traefikingress port-forward deployment/traefik 9000:9000
kubectl -n traefikingress port-forward $(kubectl -n traefikingress get pods --selector "app.kubernetes.io/name=traefik" --output=name) 9000:9000
```

* view service
`kubectl -n traefikingress describe service traefik`
`kubectl -n traefikingress get service traefik`
`kubectl -n traefikingress get pod`
