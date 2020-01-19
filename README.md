# 1. Purpose

Docker image for testing Ansible with molecule in a Fedora container.

<!-- TOC -->

- [1. Purpose](#1-purpose)
  - [1.1. Docker image](#11-docker-image)
  - [1.2. Systemd](#12-systemd)
  - [1.3. Building the image](#13-building-the-image)
  - [1.4. Tag and push the image for use with Dockerhub](#14-tag-and-push-the-image-for-use-with-dockerhub)
  - [1.5. Run a container](#15-run-a-container)
  - [1.6. Debug](#16-debug)
  - [1.7. Testing with Travis CI](#17-testing-with-travis-ci)

<!-- /TOC -->

## 1.1. Docker image

The image is build on top of the existing Fedora Docker image (provided by Fedora).  
It is prepared with **systemd** enabled and ready for testing Ansible roles with Molecule.

Used source image labels:

- **fedora:latest**
- **fedora:31**
- **fedora:30**
- **fedora:29**
- **fedora:28**

Produced image labels:

- **caseraw/ansible-molecule-fedora:latest**
- **caseraw/ansible-molecule-fedora:31**
- **caseraw/ansible-molecule-fedora:30**
- **caseraw/ansible-molecule-fedora:29**
- **caseraw/ansible-molecule-fedora:28**

> Source: <https://hub.docker.com/_/fedora/>

## 1.2. Systemd

In order for **systemd** to work, the following is provided to use in the Dockerfile when building.

```shell
RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == \
systemd-tmpfiles-setup.service ] || rm -f $i; done); \
rm -f /lib/systemd/system/multi-user.target.wants/*;\
rm -f /etc/systemd/system/*.wants/*;\
rm -f /lib/systemd/system/local-fs.target.wants/*; \
rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
rm -f /lib/systemd/system/basic.target.wants/*;\
rm -f /lib/systemd/system/anaconda.target.wants/*;

VOLUME ["/sys/fs/cgroup"]

CMD ["/usr/sbin/init"]
```

> As documented on the site: <https://hub.docker.com/_/fedora/>

## 1.3. Building the image

```shell
docker build --rm -t local/ansible-molecule-fedora:latest .
```

> Make sure to use the correct and desired tag.

## 1.4. Tag and push the image for use with Dockerhub

```shell
docker tag local/ansible-molecule-fedora:latest caseraw/ansible-molecule-fedora:latest

docker push caseraw/ansible-molecule-fedora:latest
```

> Make sure to use the correct and desired tag.

## 1.5. Run a container

```shell
docker run -ti --privileged=true -v /sys/fs/cgroup:/sys/fs/cgroup:ro local/ansible-molecule-fedora:latest
```

> Run with interactive TTY

```shell
docker run -d --privileged=true -v /sys/fs/cgroup:/sys/fs/cgroup:ro local/ansible-molecule-fedora:latest
```

> Run as daemon

## 1.6. Debug

```shell
docker exec -it <CONTAINER NAME/ID> bash
```

## 1.7. Testing with Travis CI

If needed this image can be tested and if desired pushed to the Docker hub using Travis CI.

The following can be used as an example in the **.travis.yml** file.

```yml
---
dist: bionic

language: python
python:
  - "2.7"

services:
  - docker

install:
  - docker build --rm -t local/ansible-molecule-fedora:latest .
  - docker images

script:
  - docker run -d --privileged=true -v /sys/fs/cgroup:/sys/fs/cgroup:ro local/ansible-molecule-fedora:latest
  - docker run -it --privileged=true -v /sys/fs/cgroup:/sys/fs/cgroup:ro local/ansible-molecule-fedora:latest /bin/bash -c 'uname -a'
  - docker run -it --privileged=true -v /sys/fs/cgroup:/sys/fs/cgroup:ro local/ansible-molecule-fedora:latest /bin/bash -c 'cat /etc/*release'
  - docker run -it --privileged=true -v /sys/fs/cgroup:/sys/fs/cgroup:ro local/ansible-molecule-fedora:latest /bin/bash -c 'sudo -l'
  - docker run -it --privileged=true -v /sys/fs/cgroup:/sys/fs/cgroup:ro local/ansible-molecule-fedora:latest /bin/bash -c 'python --version'
  - docker ps -a

after_success:
  - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
  - docker tag local/ansible-molecule-fedora:latest caseraw/ansible-molecule-fedora:latest
  - docker push $DOCKER_USERNAME/ansible-molecule-fedora:latest
...
```
