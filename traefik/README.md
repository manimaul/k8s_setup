```
helm install --dry-run traefik traefik/traefik > traefik/traefik.yml
<edit traefik.yml>
cat ./traefik/traefik.yml | linkerd inject - | kubectl apply -f -
```

* view logs
```
kubectl -n traefikingress get pods
kubectl -n traefikingress logs traefik-6675647c64-294xj -c traefik -f
```

* port forward into dashboard
`kubectl -n traefikingress port-forward deployment/traefik 8080:9000`

* view service
`kubectl -n traefikingress describe service traefik`
`kubectl -n traefikingress get service traefik`
`kubectl -n traefikingress get pod`
