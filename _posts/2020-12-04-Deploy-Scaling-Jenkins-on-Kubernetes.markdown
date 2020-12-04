---
layout: post
title:  "Deploy Scaling Jenkins Master-Slave on Kubernetes Cluster"
date:   2020-12-04 13:38:00 -0000
categories: kubernetes jenkins ci/cd
---
# Create Working Directory  
```
mkdir /home/ubuntu/jenkins-k8s
cd /home/ubuntu/jenkins-k8s
```  
# Create Dockerfile Jenkins Master
```
vi Dockerfile-jenkins-master 

FROM jenkins/jenkins:lts

# Plugins for better UX (not mandatory)
RUN /usr/local/bin/install-plugins.sh ansicolor
RUN /usr/local/bin/install-plugins.sh greenballs

# Plugin for scaling jenkins slaves
RUN /usr/local/bin/install-plugins.sh kubernetes

USER jenkins
```  

## Build and Push New Master Image to Docker Registry
```
sudo docker build -f Dockerfile-jenkins-master -t takatux/jenkins-master-scaling .
sudo docker push takatux/jenkins-master-scaling:latest
```

# Create PVC for jenkins master  
```
vi jenkins-volume.yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-pvc
spec:
  storageClassName: managed-nfs-storage # your storageclass name
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 8Gi
---
kubectl apply -f jenkins-volume.yaml
```  

# Create Jenkins Master Deployment
```
vi deployment.yml 
---
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: jenkins
  name: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
      - env:
        - name: JAVA_OPTS
          value: -Djenkins.install.runSetupWizard=false
        name: jenkins
        image: takatux/jenkins-master-scaling
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
          name: http-port
          protocol: TCP
        - containerPort: 50000
          name: jnlp-port
          protocol: TCP
        volumeMounts:
        - mountPath: /var/jenkins_home
          name: jenkins-home
      volumes:
      - name: jenkins-home
        persistentVolumeClaim:
           claimName: jenkins-pvc
```

# Create Service for Jenkins Master URL and Jenkins Slave tunnel
```
vi service.yml 
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 8080
  selector:
    app: jenkins
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins-jnlp
spec:
  type: ClusterIP
  ports:
    - port: 50000
      targetPort: 50000
  selector:
    app: jenkins
---

kubectl apply -f service.yml
```

# Configure Cloud  
## Add new cloud  
> Manage Jenkins >> Manage Nodes and Clouds >> Configure Clouds >> Add New Cloud >> Kubernetes  
The configuration looks like this.  
### Kubernetes Cluster Configuration
- <img src="/images/scaling-jenkins-1.png" alt="sequel pro" class="img-responsive"/>  
- <img src="/images/scaling-jenkins-2.png" alt="sequel pro" class="img-responsive"/>  
### Add the k8s config file
- <img src="/images/scaling-jenkins-3.png" alt="sequel pro" class="img-responsive"/>  
### Pod Template Configuration
- <img src="/images/scaling-jenkins-4.png" alt="sequel pro" class="img-responsive"/>  

## Setup Jenkins Master 
> Manage Jenkins >> Manage Nodes and Clouds >> Master >> set `# of executors` to 0  

# Test Freestyle Project  
> New Item >> Freestyle Project >> OK  
> General >> Check Restrict where this project can be run >> Label Exp: jenkins-slave-alpine  
> Build triggers >> Execute Shell >> Command : hostname  
> Save  
> Build now

## While the build is running check if the slave pod is running
```
ubuntu@al-master1:~$ kubectl get pods
NAME                                      READY   STATUS    RESTARTS   AGE
jenkins-5f864c95b6-qsmmc                  1/1     Running   0          2d4h
jenkins-slave-alpine-98rkx                2/2     Running   0          10s
nfs-client-provisioner-6877696cb4-tzplv   1/1     Running   0          6d22h
```
  