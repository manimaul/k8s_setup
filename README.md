# K8S Setup

We will start with a single Linode Cluster in Femont, CA and later expand to multiple clusters as needed. DNS will be provided by AWS Route53 so that we can do latency based load balancing on the multi-cluster.

Each cluster will use a Linode NodeBalancer pointing to a Linkerd injected Traefik Ingress Controller.

## Cluster 1.17

3 node cluster: 

|Plan|Monthly|Hourly|RAM|CPUs|Storage|
|----|-------|------|---|----|-------|
|2GB |$10    |$0.015|2GB|1   |50 GB  |

## NodeBalancer

* Nothing special here just service type LoadBalancer for Traefik Service

## LinkerD stable-2.8.1

* install locally 
`brew install linkerd`

* install in cluster 
`linkerd install | kubectl apply -f -`

* validate install
`linkerd check'

* see installed
`kubectl -n linkerd get deploy`

* view dashboard
`linkerd dashboard`

## Traefik

[README.md](traefik/README.md)

## Github Container Registry

todo: (WK)

## Route 53

todo: (WK)