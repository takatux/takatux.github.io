---
layout: post
title:  "Deploy HA Kubernetes Using Kubespray"
date:   2020-09-02 08:01 -0000
categories: orchestration high-availablity
---
## Install Required Packages
```
apt-get install python3-netaddr
pip3 install jinja2
vi /etc/sysctl.conf
...
net.ipv4.ip_forward=1
...
```
## Clone Kubespray Repository and Setup the Environment
```
git clone https://github.com/kubernetes-sigs/kubespray

cp -rfp inventory/sample inventory/cluster01
declare -a IPS=(10.0.0.51 10.0.0.52 10.0.0.53)
CONFIG_FILE=inventory/cluster01/inventory.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```

## Deploy kube cluster
```
ansible-playbook -i inventory/cluster01/inventory.yaml cluster.yml
``` 

## Check if the cluster is running
```
root@al-node01:~/kubespray# kubectl get pods --all-namespaces                                                                                                                        
NAMESPACE     NAME                                          READY   STATUS    RESTARTS   AGE                                                                                         
kube-system   calico-kube-controllers-659858f6cb-58cbh      1/1     Running   0          53s                                                                                         
kube-system   calico-node-4bfkf                             1/1     Running   1          70s                                                                                         
kube-system   calico-node-dk79l                             1/1     Running   1          70s                                                                                         
kube-system   calico-node-dmhlw                             1/1     Running   1          70s                                                                                         
kube-system   coredns-dff8fc7d-2th7j                        1/1     Running   0          42s                                                                                         
kube-system   coredns-dff8fc7d-mpfs6                        1/1     Running   0          38s                                                                                         
kube-system   dns-autoscaler-66498f5c5f-427w9               1/1     Running   0          40s                                                                                         
kube-system   kube-apiserver-node1                          1/1     Running   0          2m45s                                                                                       
kube-system   kube-apiserver-node2                          1/1     Running   0          2m12s                                                                                       
kube-system   kube-controller-manager-node1                 1/1     Running   0          2m45s                                                                                       
kube-system   kube-controller-manager-node2                 1/1     Running   0          2m12s                                                                                       
kube-system   kube-proxy-mxz6p                              1/1     Running   0          83s                                                                                         
kube-system   kube-proxy-nch7h                              1/1     Running   0          2m34s                                                                                       
kube-system   kube-proxy-sds86                              1/1     Running   0          2m23s                                                                                       
kube-system   kube-scheduler-node1                          1/1     Running   0          2m45s                                                                                       
kube-system   kube-scheduler-node2                          1/1     Running   0          2m12s                                                                                       
kube-system   kubernetes-dashboard-6f6cb6558c-j9rfw         1/1     Running   0          37s                                                                                         
kube-system   kubernetes-metrics-scraper-54fbb4d595-rxkwg   1/1     Running   0          37s                                                                                         
kube-system   nginx-proxy-node3                             1/1     Running   0          82s                                                                                         
kube-system   nodelocaldns-gvqk4                            1/1     Running   0          39s                                                                                         
kube-system   nodelocaldns-lp2kh                            1/1     Running   0          39s                                                                                         
kube-system   nodelocaldns-p82pt                            1/1     Running   0          39s 
```

## Test create a Deployment
```
kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
vi nginx-svc.yml
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30007
---

kubectl apply -f nginx-svc.yml
```

## Check if deployment and service are running
```
kubectl get deployments
kubeclt get svc
kubectl get pods
```
