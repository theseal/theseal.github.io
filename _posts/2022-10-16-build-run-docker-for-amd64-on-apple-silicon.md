---
layout: post
title:  "Build/run Docker for amd64 on Apple silicon"
date:   2022-10-16
---

…without Rosetta!

## Prerequisite
* Make sure that [Docker Desktop](https://www.docker.com/products/docker-desktop/) is stop (or uninstalled)
* Install [Colima](https://github.com/abiosoft/colima) and docker
```
brew install colima docker
```

* Start Colima (add --edit if you would like to modify the virtual machine)
```
colima start
```

## Run
```
docker run --platform=linux/amd64 -it --rm alpine /bin/uname -a
Linux 6b24a487230e 5.15.68-0-virt #1-Alpine SMP Fri, 16 Sep 2022 06:29:31 +0000 x86_64 Linux
```

## Build
```
docker build --platform=linux/amd64 -t path/project:latest .
```

## Push
As usual 🙃

Client certificates can be stored in `~/.docker/certs.d/<fqdn>/`.
