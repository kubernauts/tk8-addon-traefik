# TK8 addon - Traefik

## What are TK8 addons?

- TK8 add-ons provide freedom of choice for the user to deploy tools and applications without being tied to any customized formats of deployment.
- Simplified deployment process via CLI (will also be available via TK8 web in future).
- With the TK8 add-ons platform, you can also build your own add-ons.

## What is Traefik?

Traefik is a modern HTTP reverse proxy and load balancer that makes deploying microservices easy. Traefik integrates with your existing infrastructure components ([Docker](https://www.docker.com/), [Swarm mode](https://docs.docker.com/engine/swarm/), [Kubernetes](https://kubernetes.io/), [Marathon](https://mesosphere.github.io/marathon/), [Consul](https://www.consul.io/), [Etcd](https://etcd.io/), [Rancher](https://rancher.com/), [Amazon ECS](https://aws.amazon.com/ecs/), ...) and configures itself automatically and dynamically.

## Prerequisites

RBAC must be enabled on the Kubernetes Cluster.

## Get Started

You can install Traefik on the Kubernetes cluster via TK8 addons functionality.

What do you need:
- tk8 binary

## Deploy Traefik on the Kubernetes Cluster

Run:
```
$ tk8 addon install traefik
Search local for traefik
check if provided a url
Search addon on kubernauts space.
Cloning into 'traefik'...
Install traefik
execute main.sh
Creating main.yaml
add  ./traefik-config/01-traefik-rbac.yaml
add  ./traefik-config/02-traefik-daemonset.yaml
apply traefik/main.yml
clusterrole.rbac.authorization.k8s.io/traefik-ingress-controller created
clusterrolebinding.rbac.authorization.k8s.io/traefik-ingress-controller created
serviceaccount/traefik-ingress-controller created
daemonset.extensions/traefik-ingress-controller created
service/traefik-ingress-service created
traefik installation complete
```
This command will clone https://github.com/kubernauts/tk8-addon-traefik repository locally and deploy traefik.

This command also creates:
- RBAC rules essential for allowing access to resources traefik needs
- Traefik is deployed as a daemonset
- A Traefik service is also created

## Creating an Ingress for exposing Traefik web UI

We'll start by creating a Service and an Ingress resource that will expose the Traefik web UI. Create a YAML file traefik-ing-web.yaml with below contents:
```
apiVersion: v1
kind: Service
metadata:
  name: traefik-web-ui
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
  - name: web
    port: 80
    targetPort: 8080
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: traefik-web-ui
  namespace: kube-system
spec:
  rules:
  - host: traefik-ui.local
    http:
      paths:
      - path: /
        backend:
          serviceName: traefik-web-ui
          servicePort: web
```

Create the resources by running:
```
$ kubectl apply -f traefik-ing-web.yaml
service/traefik-web-ui created
ingress.extensions/traefik-web-ui created
```

Verify if the ingress was created correctly:
```
$ kubectl get ing -n kube-system
NAME             HOSTS              ADDRESS                                                                      PORTS   AGE
traefik-web-ui   traefik-ui.local   84.200.100.197,84.200.100.199,84.200.100.201,84.200.100.203,84.200.100.205   80      21m
```

Add an entry to your /etc/hosts for the traefik-ui.local URL with the value mentioned in ADDRESS field. Open traefik-ui.local in your browser and you should be greeted with the Traefik web UI.

For more examples on name-based routing, HTTP basic authentication, TLS, visit - https://docs.traefik.io/user-guide/kubernetes/

## Uninstall Traefik

For removing Traefik from your cluster, we can use TK8 addon's **destroy** functionality. Run:
```
$ tk8 addon destroy traefik
Search local for traefik
Addon traefik already exist
Found traefik local.
Destroying traefik
execute main.sh
Creating main.yaml
add  ./traefik-config/01-traefik-rbac.yaml
add  ./traefik-config/02-traefik-daemonset.yaml
delete traefik from cluster
clusterrole.rbac.authorization.k8s.io "traefik-ingress-controller" deleted
clusterrolebinding.rbac.authorization.k8s.io "traefik-ingress-controller" deleted
serviceaccount "traefik-ingress-controller" deleted
daemonset.extensions "traefik-ingress-controller" deleted
service "traefik-ingress-service" deleted
traefik destroy complete
```
