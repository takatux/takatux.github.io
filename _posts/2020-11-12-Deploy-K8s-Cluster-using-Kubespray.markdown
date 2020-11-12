---
layout: post
title:  "Deploy Kubernetes Using Kubespray"
date:   2020-11-12 09:00 -0000
categories: orchestration high-availablity
---

Kubespray is a composition of Ansible playbooks, inventory, provisioning tools, and domain knowledge for generic OS/Kubernetes clusters configuration management tasks. (kubernetes.io)  
I deployed kube cluster on my kvm.  Using 4 servers which 2 servers as masters and 2 workers. 
  
## Clone Repo and Install Packages
> git clone https://github.com/kubernetes-sigs/kubespray  
 cd kubespray/  
> sudo apt-get install python3-netaddr python3-pip  
sudo pip3 install -r requirements.txt  
sudo echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf  

## Setup Inventory file
> cp -rfp inventory/sample inventory/cluster01  
vi inventory/cluster01/inventory.ini  
```
[all]
al-master1 ansible_host=10.80.80.41
al-master2 ansible_host=10.80.80.42
al-worker1 ansible_host=10.80.80.43
al-worker2 ansible_host=10.80.80.44

[kube-master]
al-master[1:2]

[etcd]
al-master[1:2]
al-worker1

[kube-node]
al-worker[1:2]

[k8s-cluster:children]
kube-master
kube-node
```
> ansible -m ping -i inventory/cluster01/inventory.ini --become --become-user=root  all  

## Setup Group Vars
vi inventory/cluster01/group_vars/all/all.yml
```
---
etcd_data_dir: /var/lib/etcd
etcd_kubeadm_enabled: false
bin_dir: /usr/local/bin
loadbalancer_apiserver_localhost: true
loadbalancer_apiserver_type: haproxy  # valid values "nginx" or "haproxy"
loadbalancer_apiserver_port: 6443
loadbalancer_apiserver_healthcheck_port: 8081
no_proxy_exclude_workers: false
```
## Setup Network plugin in k8s-cluster.yaml
vi inventory/cluster01/group_vars/k8s-cluster/k8s-cluster.yml
```
...
kube_network_plugin: flannel
...
```
## Deploy Kubernetes Cluster
> ansible-playbook -i inventory/cluster01/hosts.yaml  --become --become-user=root cluster.yml

## Verify Deployment
> kubectl get nodes  
kubectl get pods --all-namespaces 

### Verify Deployment
> kubectl create deployment supermario-web --image=pengbai/docker-supermario --port=8080  
kubectl expose deployment supermario-web --type=NodePort --name=supermario-web  
kubectl get svc  

### Check on your browser
```
http://{master-ip}:{NodePort}
```
