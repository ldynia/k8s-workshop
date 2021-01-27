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

# Kubernetes Architecture

![Architecture](https://raw.githubusercontent.com/ldynia/k8s-workshop/master/img/pyramid.png)

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

## Setup TLS keys

```bash
$ ssh-keygen -t rsa -b 4096 -N ''
$ ls -l ~/.ssh
-rw------- 1 user user 2610 Nov  3 13:50 id_rsa
-rw-r--r-- 1 user user  574 Nov  3 13:50 id_rsa.pub
-rw-r--r-- 1 user user 1776 Jan 27 10:25 known_hosts
```

## Setting vCluster with Vagrant

```bash
$ cd ~/k8workshop
$ git clone git@github.com:ldynia/vagrant-ansible-k8-centos.git vlab/
$ cd vlab/vagrant/

$ vagrant up
$ vagrant status
$ vagrant ssh master
```

## Provisioning Cluster With Ansible

Add IPs of virtual cluster to `/etc/hosts`

```
# /etc/hosts
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

![kubernetes constructs](https://github.com/ldynia/k8s-workshop/raw/master/img/pyramid.png)

## docker vs kubernetes; container vs pod

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
```
$ kubectl get pods
$ kubectl describe pods alpinus
$ kubectl get pods alpinus -o yaml
$ kubectl get pods alpinus -o yaml > alpinus.pod.yaml
$ kubectl delete pod alpinus
$ kubectl apply -f alpinus.pod.yaml
```

## HA Redis

```bash
$ helm repo add stable https://charts.helm.sh/stable
$ helm repo update
$ helm search repo stable/redis
$ helm install redis-ha stable/redis-ha
$ helm ls
```
