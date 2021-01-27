# Description

This repository is designed to demonstrate capabilities of [kubernetes](https://kubernetes.io/).

# Installation

Requirements

* [Docker](https://docs.docker.com/)
* [Docker-Comose](https://docs.docker.com/compose/install/)
* [Docker Hub Account](https://hub.docker.com/)
* [Appllication](https://github.com/ldynia/flask-k8s)
* [Virtual Kubernetes Cluster](https://github.com/ldynia/vagrant-vlab)

# History

At the moment of writing Kubernetes is 6 years old. It originated from Google's **7th** Project [Borg](https://www.gcppodcast.com/post/episode-46-borg-and-k8s-with-john-wilkes/) and name was inspired by Star Trek's [Ex-Borg](https://en.wikipedia.org/wiki/Seven_of_Nine).

# Kubernetes

## Application Flow

![app lifecycle](https://raw.githubusercontent.com/ldynia/k8s-workshop/master/img/flow.png)

## Architecture

![architecture](https://miro.medium.com/max/2400/1*HXbT0c4Q5XaiCIp6y3VMvw.png)

## Constructs

![kubernetes constructs](https://raw.githubusercontent.com/ldynia/k8s-workshop/master/img/pyramid.png)

# Application - Flask

## Running application

```bash
$ cd ~
$ mkdir k8workshop
$ cd k8workshop
$ git clone git@github.com:ldynia/flask-k8s.git
$ cd flask-k8s/

$ docker-compose build
$ docker-compose up

$ docker ps | grep flask
$ docker exec flask-k8s-app hostname
$ docker exec flask-k8s-app flask routes
```

## Building application

```bash
$ cd app
$ docker build --tag ldynia/k8s:latest --file ../docker/Dockerfile --build-arg APP_COLOR=green .

$ docker images | grep ldynia

$ docker run -d ldynia/k8s:latest
```

## Distributing application to public Registry

```bash
$ docker login
$ docker push ldynia/k8s:latest
```

# Virtual Cluster - VirtualBox

## Setup TLS

```bash
$ ssh-keygen -t rsa -b 4096 -N ''
$ ls -l ~/.ssh
-rw------- 1 user user 2610 Nov  3 13:50 id_rsa
-rw-r--r-- 1 user user  574 Nov  3 13:50 id_rsa.pub
-rw-r--r-- 1 user user 1776 Jan 27 10:25 known_hosts
```

## Setting vCluster - Vagrant

```bash
$ cd ~/k8workshop
$ git clone git@github.com:ldynia/vagrant-ansible-k8-centos.git vlab/
$ cd vlab/vagrant/

$ vagrant up
$ vagrant status
$ vagrant ssh master
```

## Provisioning vCluster - Ansible

Add IPs of cluster nodes to `/etc/hosts`

```
192.168.234.200 k8client
192.168.234.230 k8master
192.168.234.231 k8worker1
192.168.234.232 k8worker2
192.168.234.233 k8worker3
```

Make suer that you can ssh without any keyboard interaction. In case of offending keys you can use this commands `ssh-keygen -R` and `sed`.

```bash
{
    ssh-keygen -f ~/.ssh/known_hosts -R "k8client"
    ssh-keygen -f ~/.ssh/known_hosts -R "k8master"
    ssh-keygen -f ~/.ssh/known_hosts -R "k8worker1"
    ssh-keygen -f ~/.ssh/known_hosts -R "k8worker2"
}
{
    sed -i '12d' ~/.ssh/known_hosts
    sed -i '12d' ~/.ssh/known_hosts
    sed -i '12d' ~/.ssh/known_hosts
    sed -i '12d' ~/.ssh/known_hosts
}
```

```bash
$ ssh vagrant@k8client
$ ssh vagrant@k8master
$ ssh vagrant@k8worker1
$ ssh vagrant@k8worker2
```

Ansible

```bash
$ cd ~/k8workshop/vlab/ansible

$ ansible -m ping all
$ ansible -m shell -a 'hostname' all
$ ansible -m shell -a 'df -h' all
```

Ansible Playbook

```bash
$ ansible-playbook playbook-common.yml
$ ansible-playbook playbook-kubernetes.yml
```

# Kubernetes

## Namespaces

```bash
$ docker run --name c1 busybox sleep 1d
$ docker run --name c2 busybox sleep 1d
$ docker exec c1 ps
$ docker exec c2 ps
$ ps aux | grep sleep
$ docker rm c2
$ docker run --name c2 --pid container:c1 busybox sleep 1d
```

## Docker vs Kubernetes; Container vs Pod

Container

```bash
$ docker run -d --name alpinus alpine sleep 3600
$ docker exec alpinus ps
$ docker exec -it alpinus ash
```

Pod

```bash
$ kubectl run alpinus --image alpine --command -- sleep 3600
$ kubectl exec alpinus -- ps
$ kubectl exec -it alpinus -- ash
```

Inspecting Pods

```bash
$ kubectl get pods
$ kubectl describe pods alpinus
$ kubectl get pods alpinus -o yaml
$ kubectl get pods alpinus -o yaml > alpinus.pod.yaml
$ kubectl delete pod alpinus
$ kubectl apply -f alpinus.pod.yaml
```

# Sets

Manages the deployment and scaling of a Pod

## Daemon Set

A Daemon Set ensures that all (or some) Nodes run a copy of one Pod. 

As nodes are added to the cluster, Pod is added to them. As nodes are removed from the cluster, Pod is garbage collected.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      #tolerations:
      #- key: node-role.kubernetes.io/master
      #  effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
```

## Stateful Set

Stateful Set provides guarantees about the **ordering** and **uniqueness** of Pods, by maintains a sticky identity for each of its Pods. Pods in Stateful Set are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-ss
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web-ss
spec:
  selector:
    matchLabels:
      app: nss
  serviceName: nginx-ss
  replicas: 3
  template:
    metadata:
      labels:
        app: nss
    spec:
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
```

```bash
$ kubectl get pods -l app=nss
$ kubectl exec web-0 -- hostname
$ kubectl exec web-0 -- curl web-0.nginx.default.svc.cluster.local
```

## Replica Set

Purpose of ReplicaSet is to maintain a stable set of replica Pods running at any given time.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: demo-rs
  labels:
    run: drs
spec:
  replicas: 3
  selector:
    matchLabels:
      run: drs
  template:
    metadata:
      labels:
        run: drs
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```

# Deployment

Deployment is a higher-level concept that manages ReplicaSets and provides declarative updates to Pods and ReplicaSets. 

You describe a desired state of an application in a Deployment, and the Deployment Controller changes current state to the desired state at a controlled rate.

**It's recommend using Deployments instead of ReplicaSets, unless you require custom update orchestration or don't require updates at all.**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

```bash
$ kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 --record
$ kubectl rollout status deployment nginx-deployment 
$ kubectl rollout history deployment nginx-deployment 
$ kubectl rollout undo deployment.v1.apps/nginx-deployment --to-revision=1
```

## HA Redis

```bash
$ helm repo add stable https://charts.helm.sh/stable
$ helm repo update
$ helm search repo stable/redis
$ helm install redis-ha stable/redis-ha
$ helm ls
```
