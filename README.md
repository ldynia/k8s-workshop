# Description

This repository is designed to demonstrate capabilities of [kubernetes](https://kubernetes.io/).

# Installation

Requirements

* [Appllication](https://github.com/ldynia/flask-k8s)
* [Virtual Kubernetes Cluster](https://github.com/ldynia/vagrant-vlab)
* [Docker Hub Account](https://hub.docker.com/)
* [Docker-Comose](https://docs.docker.com/compose/install/)

# History

At the moment of writing Kubernetes is 6 years old. It originated from Google's **7th** Project [Borg](https://www.gcppodcast.com/post/episode-46-borg-and-k8s-with-john-wilkes/) and name was inspired by Star Trek's [Ex-Borg](https://en.wikipedia.org/wiki/Seven_of_Nine).

# Kubernetes Architecture

![Architecture](https://miro.medium.com/max/2400/1*HXbT0c4Q5XaiCIp6y3VMvw.png)

# Application - Flask

### Running application

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


### Building application

```bash
$ cd app
$ docker build --tag ldynia/k8s:red --file ../docker/Dockerfile --build-arg APP_COLOR=red .
$ docker build --tag ldynia/k8s:blue --file ../docker/Dockerfile --build-arg APP_COLOR=blue .
$ docker build --tag ldynia/k8s:latest --file ../docker/Dockerfile --build-arg APP_COLOR=latest .
$ docker images | grep ldynia

$ docker run -d ldynia/k8s:red
$ docker run -d ldynia/k8s:blue
$ docker run -d ldynia/k8s:latest
```

### Distributing application to public Registry

```bash
$ docker login
$ docker push ldynia/k8s:red
$ docker push ldynia/k8s:blue
$ docker push ldynia/k8s:latest
```

