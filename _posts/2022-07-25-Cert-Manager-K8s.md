---
layout: post
title:  "Nginx-Ingress and Cert-Manager Issuer in Kubernetes"
date:   2022-07-25 02:15:00 -0000
categories: devsecops kubernetes
---

Use Case :  
Application need to be able to be access from public. There are a bunch of application that have been deployed in kubernetes. Each application service has specific hostname that identified by domain. So, we need an ingress gateway to route each domain to particular service. In this case we are using [Nginx-Ingress](https://github.com/kubernetes/ingress-nginx).  

The connection between user and server also must be a secure connection. So we need to enable TLS to be able to use HTTPS protocol.  In this case, we decided to use [Cert-Manager](https://cert-manager.io/v0.14-docs) as certificate issuer.  

## Nginx-Ingress Deployment

Referring to the [this documentation](https://kubernetes.github.io/ingress-nginx/deploy/), you can simply apply this manifest to deploy Ingress-Nginx.  

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.1/deploy/static/provider/cloud/deploy.yaml
```

Check that all pods are running and create an ingress to ensure Nginx-Ingress is working properly.  

## Cert-Manager Deployment
> *please notice that cert-manager require you to register your domain first to verify so your certificate will be valid*
>  

Check [this refference](https://cert-manager.io/v0.14-docs) for more details.  

Cert-Manager can be deployed by applying this manifest.  

```
kubectl create namespace cert-manager
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.14.3/cert-manager.yaml
```

Check if cert-manager already running.  

```
kubectl get pods -n cert-manager
```

Result will be as shown below.  

### Create Cluster Issuer

Cluster issuer is an issuer that can be used by all namespaces in the cluster. Below is the manifest to create cluster issuer.  

```
cat <<EOF > cluster-issuer.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: example@katondu.net
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod
    # Enable the HTTP-01 challenge provider
    solvers:
    - http01:
        ingress:
          class: nginx
EOF
```

Apply the manifest  

```
kubectl apply -f cluster-issuer.yaml
```

> *you can also create namespace issuer instead of clusterIssuer if need*
> 

### Utilize Cluster Issuer

Basically, we need to define the issuer in ingress manifest so that ingress can request a valid certificate to the Cert-Manager.  

Here is the ingress for example :  

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-katondu-ingress
  namespace: katondu
  annotations:
    kubernetes.io/ingress.class: nginx
    ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - example.katondu.net
    secretName: example-katondu-tls
  rules:
  - host: example.katondu.net
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: example-katondu
            port:
              number: 80
```

Please notice there are two main section :  

1. annotations inside metadata  
2. tls inside spec  

Here is the template of nginx-ingress with TLS-enabled. You can change the value inside bracket [] :  

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: [servicename]-ingress
  namespace: [namespace of the service]
  annotations:
    kubernetes.io/ingress.class: nginx
    ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - [domain name]
    secretName: [servicename]-letsencrypt
  rules:
  - host: [domain name]
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: [service-name]
            port:
              number: [port of the service]
```

Please ensure you have create the respective domain records in Route53 before running this step  

explaination :  
 
- metadata.name = name of the ingress by using your service with ingress prefix  
- metadata.namespace = namespace where the service resides  
- metadata.annotations = you still can add more annotations but   annotations defined in this template is mandatory.  
- spec.tls.hosts[] = the domain name that will be used for certificate request  
- rules = routing configuration  